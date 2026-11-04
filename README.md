# PyOlympiad

PyOlympiad is a browser-based Python coding environment built with **Flask**. It is designed for students to write and execute Python programs through a web interface.

## Features

- Browser-based Python code editor.
- Run Python code and view standard output and errors.
- Accept standard input for programs that use `input()`.
- Execution timeout to stop programs that run too long.
- Per-run temporary working directories.
- Configurable code, input, and output size limits.
- Per-IP rate limiting and a limit on concurrent executions.
- Optional site-wide HTTP Basic Authentication.
- Health-check endpoint for deployment monitoring.
- UTF-8 output support, including Unicode characters and emoji.

## Technology Stack

- Python
- Flask 3.0.3
- Gunicorn 22.0.0
- HTML, CSS, and JavaScript

## Project Structure

```text
PyOlympiad-main/
├── app.py
├── config.py
├── requirements.txt
├── templates/
│   ├── index.html
│   ├── base.html
│   └── editor.html
└── README.md
```

`app.py` contains the Flask application and Python execution endpoint.  
`config.py` contains application and execution settings.  
`templates/index.html` provides the main coding interface.

## Requirements

- Python 3.9 or newer (Python 3.10+ recommended).
- pip.
- A working Python interpreter available to the Flask process.

Install the dependencies:

```bash
pip install -r requirements.txt
```

## Run Locally

1. Extract or clone the project.
2. Open a terminal in the project directory.
3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

4. Start the application:

   ```bash
   python app.py
   ```

5. Open this address in your browser:

   ```text
   http://127.0.0.1:5000
   ```

The default host is `0.0.0.0` and the default port is `5000`. The application also reads the `PORT` environment variable, which is commonly provided by hosting platforms.

## Configuration

Application settings are defined in `config.py`. Most deployment settings can be overridden with environment variables.

| Setting | Environment variable | Default | Description |
|---|---|---:|---|
| Secret key | `SECRET_KEY` | `dev-only-change-me` | Flask session/security key |
| Debug mode | `DEBUG` | `false` | Enables Flask debug mode only when set to `true` |
| Host | `HOST` | `0.0.0.0` | Local server bind address |
| Port | `PORT` | `5000` | Server port |
| Execution timeout | `CODE_TIMEOUT` | `5` | Maximum wall-clock execution time in seconds |
| CPU limit | `CPU_LIMIT` | `10` | POSIX CPU-time limit in seconds |
| Memory limit | `MEMORY_MB` | `256` | POSIX address-space limit in MB |
| Concurrent runs | `MAX_CONCURRENT_RUNS` | `3` | Maximum simultaneous program executions |
| Rate limit | `RATE_LIMIT_MAX` | `120` | Maximum runs per IP within the configured window |
| Rate-limit window | `RATE_LIMIT_WINDOW` | `60` | Rate-limit window in seconds |
| Basic Auth username | `BASIC_AUTH_USER` | empty | Enables site-wide Basic Authentication when both credentials are set |
| Basic Auth password | `BASIC_AUTH_PASS` | empty | Password for site-wide Basic Authentication |

The application also defines fixed limits in `config.py`:

- Maximum submitted code: 20,000 characters.
- Maximum standard input: 10,000 characters.
- Maximum returned output per stream: 50,000 characters.
- Maximum file size written by a student program on POSIX: 1 MB.

For production, set a strong, unique `SECRET_KEY`, keep debug mode disabled, and configure Basic Authentication if the site should be restricted.

## Deploying with Gunicorn

The project includes Gunicorn. A typical production start command is:

```bash
gunicorn app:app
```

On a platform such as Render:

1. Create a new Web Service and connect the repository.
2. Select the Python runtime.
3. Set the build command:

   ```bash
   pip install -r requirements.txt
   ```

4. Set the start command:

   ```bash
   gunicorn app:app
   ```

5. Add any required environment variables in the hosting dashboard.
6. Ensure the service runs in a Linux environment if you rely on the POSIX resource-limit launcher.

## Health Check

The application provides:

```text
GET /health
```

It returns `ok` with HTTP status `200` and is excluded from the optional Basic Authentication check. This endpoint can be used as the hosting platform's health-check path.

## Code Execution API

The browser submits code to:

```text
POST /run
Content-Type: application/json
```

Example request:

```json
{
  "code": "name = input('Name: ')\nprint('Hello,', name)",
  "input": "Alex"
}
```

The response is JSON containing:

- `success`: whether the program completed successfully.
- `output`: captured standard output.
- `error`: captured error text or an execution error message.

The endpoint validates input sizes, applies rate and concurrency limits, runs the submitted program in a temporary directory, captures output, and removes the temporary directory after execution.

## Important Security Notice

PyOlympiad executes Python code submitted by users. Resource limits and temporary directories are useful safeguards, but **they do not make arbitrary code execution fully secure**.

Before exposing this application to the public internet:

- Run it in a dedicated, isolated container or virtual machine with no sensitive host files or credentials.
- Use a non-root operating-system account.
- Apply network egress restrictions and container-level CPU, memory, process, and filesystem limits.
- Do not mount secrets, production databases, Docker sockets, or other sensitive resources into the execution environment.
- Add monitoring and maintain the operating system and Python runtime.
- Review and test the sandbox against your threat model.

The POSIX launcher applies `resource` limits on Linux/Unix. The Windows execution path does not apply the same OS-level CPU, memory, and file-size limits. For public deployment, use a properly isolated Linux execution environment rather than relying on the Windows path.

## License

No license file was included in the supplied project archive. Add a `LICENSE` file before redistributing the project if you intend to publish it as open source.
