# Django Base Project

A modern Django 6.0.1 project template with uv dependency management, Docker containerization, PostgreSQL database, and Ruff linting.

## Features

- **Django 6.0.1** - Latest stable Django version
- **uv** - Fast Python package installer and resolver
- **Docker & Docker Compose** - Containerized development environment
- **PostgreSQL 16** - Production-ready database with persistent storage
- **django-environ** - Environment-based configuration
- **Ruff** - Fast Python linter and formatter
- **Hot Reload** - Automatic code reloading during development
- **Custom Port** - Runs on port 9000 instead of default 8000

## Prerequisites

- Docker
- Docker Compose
- uv (optional, for local IDE support and linting)

## Project Structure

```
django-base/
├── docker/              # Docker configuration
│   ├── Dockerfile       # Django container definition
│   ├── docker-compose.yml  # Multi-container orchestration
│   └── .dockerignore    # Docker build exclusions
├── config/              # Django project configuration
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py      # Main settings with environment variables
│   ├── urls.py
│   └── wsgi.py
├── manage.py            # Django management script
├── pyproject.toml       # Project dependencies and metadata
├── uv.lock              # Locked dependency versions
├── ruff.toml            # Ruff linting configuration
├── .env                 # Environment variables (not in git)
├── .env.example         # Environment variables template
└── README.md
```

## Quick Start

1. **Clone and navigate to the project**
   ```bash
   cd django-base
   ```

2. **Create environment file**
   ```bash
   cp .env.example .env
   # Edit .env and update SECRET_KEY and other variables as needed
   ```

3. **Build and start containers**
   ```bash
   docker-compose -f docker/docker-compose.yml up --build
   ```

   Or navigate to the docker directory:
   ```bash
   cd docker
   docker-compose up --build
   ```

4. **Access the application**

   Open your browser to [http://localhost:9000](http://localhost:9000)

5. **Create a superuser** (in a new terminal)
   ```bash
   docker-compose -f docker/docker-compose.yml exec web uv run python manage.py createsuperuser
   ```

## Local Development with IDE Support

While all Django commands must run inside containers, you can set up a local uv environment for IDE features like import resolution, autocomplete, and type checking in PyCharm or other IDEs.

### Setting up local uv environment (optional)

```bash
# Install dependencies locally
uv sync

# This creates a .venv directory for IDE to use
```

**Important Notes:**
- The local `.venv` is ONLY for IDE support (import resolution, autocomplete, type hints)
- All Django commands (migrate, runserver, shell, etc.) must run in Docker containers
- When you add packages with `uv add`, you must rebuild the Docker image:
  ```bash
  # Add a package locally
  uv add package-name

  # Rebuild Docker image (hot reload won't install new packages)
  docker-compose -f docker/docker-compose.yml up --build
  ```

### Running Ruff locally (without Docker)

Ruff can run locally for faster linting during development:

```bash
# Check code
uv run ruff check .

# Check and fix
uv run ruff check . --fix

# Format code
uv run ruff format .
```

## Environment Variables

Create a `.env` file in the project root with the following variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `SECRET_KEY` | Django secret key (generate with `python -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'`) | `django-insecure-...` |
| `DEBUG` | Enable debug mode | `True` or `False` |
| `ALLOWED_HOSTS` | Comma-separated list of allowed hosts | `localhost,127.0.0.1` |
| `POSTGRES_DB` | PostgreSQL database name | `django_db` |
| `POSTGRES_USER` | PostgreSQL username | `django_user` |
| `POSTGRES_PASSWORD` | PostgreSQL password | `django_password` |
| `POSTGRES_HOST` | PostgreSQL host (use `db` for Docker) | `db` |
| `POSTGRES_PORT` | PostgreSQL port | `5432` |

## Django Management Commands

All Django commands must be executed inside the Docker container:

```bash
# General pattern
docker-compose -f docker/docker-compose.yml exec web uv run python manage.py <command>
```

Common commands:
```bash
# Apply migrations
docker-compose -f docker/docker-compose.yml exec web uv run python manage.py migrate

# Create migrations
docker-compose -f docker/docker-compose.yml exec web uv run python manage.py makemigrations

# Create superuser
docker-compose -f docker/docker-compose.yml exec web uv run python manage.py createsuperuser

# Open Django shell
docker-compose -f docker/docker-compose.yml exec web uv run python manage.py shell

# Open database shell
docker-compose -f docker/docker-compose.yml exec web uv run python manage.py dbshell

# Collect static files
docker-compose -f docker/docker-compose.yml exec web uv run python manage.py collectstatic
```

## Docker Commands

```bash
# Start containers (from project root)
docker-compose -f docker/docker-compose.yml up

# Start in background
docker-compose -f docker/docker-compose.yml up -d

# Stop containers
docker-compose -f docker/docker-compose.yml down

# View logs
docker-compose -f docker/docker-compose.yml logs -f

# View logs for specific service
docker-compose -f docker/docker-compose.yml logs -f web

# Rebuild containers
docker-compose -f docker/docker-compose.yml up --build

# Remove volumes (WARNING: deletes database data)
docker-compose -f docker/docker-compose.yml down -v

# Alternative: Run from docker directory
cd docker
docker-compose up
docker-compose down
# etc.
```

## Dependency Management

When adding or updating dependencies, remember to rebuild the Docker image:

```bash
# Add a new dependency
uv add package-name

# Add a dev dependency
uv add --dev package-name

# Rebuild Docker to install new packages
docker-compose -f docker/docker-compose.yml up --build

# Show installed packages (in container)
docker-compose -f docker/docker-compose.yml exec web uv pip list
```

**Note:** Hot reload only works for code changes, not for new package installations. Always rebuild after modifying dependencies.

## Database Management

### Backup Database

```bash
docker-compose -f docker/docker-compose.yml exec db pg_dump -U django_user django_db > backup.sql
```

### Restore Database

```bash
docker-compose -f docker/docker-compose.yml exec -T db psql -U django_user django_db < backup.sql
```

### Access Database Shell

```bash
docker-compose -f docker/docker-compose.yml exec db psql -U django_user -d django_db
```

## Troubleshooting

### Port 9000 already in use

```bash
# Find process using port 9000
lsof -i :9000

# Kill the process
kill -9 <PID>

# Or change the port in docker/docker-compose.yml
```

### Database connection errors

- Verify credentials in `.env` match your configuration
- Ensure `db` service is healthy: `docker-compose -f docker/docker-compose.yml ps`
- Check logs: `docker-compose -f docker/docker-compose.yml logs db`

### Hot reload not working

- Hot reload works for code changes automatically
- For new packages, you must rebuild: `docker-compose -f docker/docker-compose.yml up --build`
- Check volume mount is correct in `docker/docker-compose.yml`

### Import errors in IDE (PyCharm, VSCode, etc.)

This means your local uv environment is missing or outdated:
```bash
# Sync local environment
uv sync

# Configure your IDE to use .venv/bin/python as the interpreter
```

### Migration errors

```bash
# Reset migrations (WARNING: deletes data)
docker-compose -f docker/docker-compose.yml down -v
docker-compose -f docker/docker-compose.yml up --build
```

### Container won't start

```bash
# View logs
docker-compose -f docker/docker-compose.yml logs web

# Remove old containers and rebuild
docker-compose -f docker/docker-compose.yml down
docker-compose -f docker/docker-compose.yml up --build
```

## Production Considerations

Before deploying to production:

1. Generate a new `SECRET_KEY`
2. Set `DEBUG=False`
3. Configure proper `ALLOWED_HOSTS`
4. Use a production-ready WSGI server (gunicorn, uwsgi)
5. Set up proper static file serving
6. Configure HTTPS
7. Use environment-specific settings
8. Set up proper logging
9. Configure database backups
10. Use managed PostgreSQL service

## License

This project is open source and available under the MIT License.
