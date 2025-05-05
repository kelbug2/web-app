# web-app
dating app for da goth girlz


# Secure Dating Web Application

This is a Django-based web application demonstrating a basic dating platform with user registration, profiles, matching, and chat features, built with security best practices in mind.

**⚠️ Disclaimer:** While built with security practices, this application is a demonstration and should be thoroughly reviewed and tested before any potential deployment. It is recommended for educational or portfolio use rather than a live production environment.

## Features

*   **User Management:**
    *   Registration with email and password.
    *   Secure login using Django's authentication system.
    *   Password hashing using Argon2 (configurable fallback to PBKDF2).
*   **User Profiles:**
    *   Editable profiles with fields like orientation, gender, age, bio, photos (placeholders), and match preferences.
*   **Matching:**
    *   Ability to "like" other users.
    *   Mutual likes result in a match.
    *   View list of matches.
*   **Messaging:**
    *   Private chat between matched users.
*   **Security & Auditing:**
    *   Audit logs for login attempts, profile updates, and messages sent.
    *   Basic role implementation (User, Moderator, Admin - defined in `UserProfile` model).
    *   CSRF protection enabled by default.
    *   Input validation via Django Forms.
    *   XSS protection via Django's default template auto-escaping.
    *   HTTPS readiness settings (via `.env`).

## Project Structure

```
/secure_dating_app
|-- .env                   # Environment variables (SECRET KEY, DB config)
|-- Dockerfile             # For building the Docker image
|-- docker-compose.yml     # For running the app and optional PostgreSQL DB with Docker
|-- manage.py              # Django management script
|-- requirements.txt       # Python dependencies
|-- setup_ubuntu.sh        # Bash script for Ubuntu setup
|-- secure_dating_project/ # Django project configuration
|   |-- settings.py
|   |-- urls.py
|   |-- wsgi.py
|   |-- ...
|-- dating_app/            # Main Django application
|   |-- models.py
|   |-- views.py
|   |-- urls.py
|   |-- forms.py
|   |-- admin.py
|   |-- migrations/
|   |-- templates/         # App-specific templates
|   |   `-- dating_app/
|   |       |-- base.html
|   |       |-- home.html
|   |       |-- register.html
|   |       |-- login.html
|   |       |-- profile_view.html
|   |       |-- profile_edit.html
|   |       |-- users_list.html
|   |       |-- matches_view.html
|   |       |-- chat_view.html
|   |       # Removed: sqli_example.html
|-- templates/             # Project-level templates (contains base.html)
|-- static/                # Project-level static files (CSS, JS)
|-- media/                 # User-uploaded files (e.g., photos)
|-- venv/                  # Python virtual environment (created by setup script)
`-- db.sqlite3             # Default SQLite database file
```

## Setup and Installation

You can set up the application locally on Ubuntu or using Docker.

### Option 1: Local Ubuntu Setup (using `setup_ubuntu.sh`)

This method uses a Python virtual environment and installs dependencies directly on your Ubuntu system (tested on Ubuntu 22.04+).

1.  **Prerequisites:**
    *   Ubuntu Linux (22.04+ recommended)
    *   `python3.11` and `python3.11-venv` installed.
        ```bash
        sudo apt update
        sudo apt install python3.11 python3.11-venv git
        ```
    *   Optional: PostgreSQL client libraries if using PostgreSQL database.
        ```bash
        sudo apt install libpq-dev
        ```

2.  **Clone the Repository (if applicable):**
    ```bash
    # git clone <repository_url>
    # cd secure_dating_app
    ```
    *(Assuming you already have the project files in `/home/ubuntu/secure_dating_app`)*

3.  **Run the Setup Script:**
    Navigate to the project directory (`/home/ubuntu/secure_dating_app`) and run:
    ```bash
    bash setup_ubuntu.sh
    ```
    This script will:
    *   Create a Python virtual environment in `./venv`.
    *   Install required packages from `requirements.txt`.
    *   Create a default `.env` file if one doesn't exist.
    *   Apply database migrations (using SQLite by default).
    *   Optionally prompt you to create a superuser.

4.  **Configure `.env`:**
    *   **CRITICAL:** Open the `.env` file (`/home/ubuntu/secure_dating_app/.env`) and set a unique, strong `DJANGO_SECRET_KEY`.
You can generate one using Python: `python3.11 -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'`
    *   Review other settings, especially database configuration if you want to use PostgreSQL.

5.  **Run the Development Server:**
    ```bash
    # Activate the virtual environment
    source venv/bin/activate

    # Run the server
    python manage.py runserver
    ```

6.  **Access the Application:**
    Open your web browser and go to `http://127.0.0.1:8000/`.

### Option 2: Docker Setup (using `docker-compose`)

This method uses Docker and Docker Compose to run the application in containers. It includes an optional PostgreSQL service.

1.  **Prerequisites:**
    *   Docker installed: [Get Docker](https://docs.docker.com/engine/install/)
    *   Docker Compose installed: [Install Docker Compose](https://docs.docker.com/compose/install/)

2.  **Clone the Repository (if applicable):**
    ```bash
    # git clone <repository_url>
    # cd secure_dating_app
    ```
    *(Assuming you already have the project files in `/home/ubuntu/secure_dating_app`)*

3.  **Configure `.env`:**
    *   Ensure the `.env` file exists in the project root (`/home/ubuntu/secure_dating_app/.env`).
    *   **CRITICAL:** Set a unique, strong `DJANGO_SECRET_KEY`.
    *   Configure database settings:
        *   **For SQLite (Default):** Keep the default `DB_ENGINE` and `DB_NAME` or leave them commented out.
        *   **For PostgreSQL:** Uncomment the PostgreSQL variables (`DB_ENGINE`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`). Set `DB_HOST=db` (the service name in `docker-compose.yml`). The `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD` in `docker-compose.yml` should match these values.

4.  **Build and Run with Docker Compose:**
    Open a terminal in the project root directory (`/home/ubuntu/secure_dating_app`) and run:
    ```bash
    docker-compose up --build
    ```
    *   `--build` forces Docker to rebuild the image if the `Dockerfile` or application code has changed.
    *   This command will build the `web` image, start the `web` service, and optionally start the `db` (PostgreSQL) service.
    *   The first time you run it, Django migrations should be applied automatically by the `web` container (check container logs). If not, run them manually (see below).

5.  **Apply Migrations & Create Superuser (if needed, manually):**
    If migrations don't run automatically or you need to create a superuser:
    *   Open another terminal.
    *   Run migrations inside the running `web` container:
        ```bash
        docker-compose exec web python manage.py migrate
        ```
    *   Create a superuser:
        ```bash
        docker-compose exec web python manage.py createsuperuser
        ```

6.  **Access the Application:**
    Open your web browser and go to `http://localhost:8000/` (or `http://<your-docker-host-ip>:8000/` if running Docker on a remote machine).

7.  **Stopping the Application:**
    Press `Ctrl+C` in the terminal where `docker-compose up` is running. To remove the containers, run:
    ```bash
    docker-compose down
    ```
    To remove containers and volumes (including database data if using the `db` service):
    ```bash
    docker-compose down -v
    ```

## Configuration (`.env` file)

The `.env` file in the project root directory controls key settings:

*   `DJANGO_SECRET_KEY`: **Required.** A unique, unpredictable value for cryptographic signing.
*   `DJANGO_DEBUG`: `True` for development (shows detailed errors), `False` for production.
*   `DJANGO_ALLOWED_HOSTS`: Comma-separated list of hosts/domains the site can serve. E.g., `localhost,127.0.0.1,.example.com`.

*   **Database:**
    *   `DB_ENGINE`: `django.db.backends.sqlite3` or `django.db.backends.postgresql`.
    *   `DB_NAME`: Path for SQLite (e.g., `db.sqlite3`) or database name for PostgreSQL.
    *   `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`: Required for PostgreSQL.

*   **Security Headers (for HTTPS):** Set these to `True` when deploying behind HTTPS.
    *   `CSRF_COOKIE_SECURE=False`
    *   `SESSION_COOKIE_SECURE=False`
    *   `SECURE_SSL_REDIRECT=False`
    *   `SECURE_HSTS_SECONDS=0`
    *   `SECURE_HSTS_INCLUDE_SUBDOMAINS=False`
    *   `SECURE_HSTS_PRELOAD=False`

**Remember to restart the Django development server or Docker containers after changing `.env` values.**

## Using the Application

1.  Register a new user account or log in.
2.  Edit your profile (`/profile/edit/`) to add details.
3.  Browse other users (`/users/`).
4.  "Like" users you are interested in.
5.  If another user likes you back, they will appear in your Matches list (`/matches/`).
6.  You can chat with your matches (`/chat/<username>/`).

## Contributing

Contributions are welcome, especially those that enhance the features or improve security following best practices.
