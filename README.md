# Task Management REST API

A secure, multi-user task management backend built with Django REST Framework. Each registered user can create, update, and track their own tasks through a set of JWT-protected endpoints — nobody can see or touch another user's data.

This is a headless API only: there is no server-rendered HTML or admin-facing dashboard beyond Django's built-in admin panel. Every piece of functionality is exposed as a JSON endpoint, documented and testable directly through an auto-generated Swagger interface.

## Stack

| Concern              | Choice                                  |
| -------------------- | ---------------------------------------- |
| Framework             | Django + Django REST Framework           |
| Authentication        | JWT (djangorestframework-simplejwt)      |
| Database               | MySQL                                    |
| Filtering & search     | django-filter                            |
| API documentation      | drf-spectacular (Swagger UI / OpenAPI 3) |
| Environment config      | python-decouple                          |

## How it works

1. A new user registers through `/api/auth/register/` with a username, email, and password.
2. The user logs in through `/api/auth/login/` and receives a pair of JWT tokens — an **access** token (short-lived, used to authorize requests) and a **refresh** token (longer-lived, used to get a new access token without logging in again).
3. Every request to a task endpoint must carry the access token in the `Authorization: Bearer <token>` header.
4. Once authorized, the user can create, list, retrieve, update, or delete tasks — but the API always filters results by the logged-in user, so one account's tasks are never visible or editable by another account.
5. Task lists can be filtered by status or priority, searched by title/description, and sorted by due date, priority, or creation time.

## Getting started

Requires Python 3.x, a running MySQL server, and pip.

```bash
pip install -r requirements.txt
```

Create a `.env` file in the project root:

```
SECRET_KEY=your_django_secret_key
DEBUG=True

DB_NAME=task_management_db
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_HOST=localhost
DB_PORT=3306
```

Create the database in MySQL, then run migrations:

```bash
python manage.py makemigrations
python manage.py migrate
```

Start the development server:

```bash
python manage.py runserver
```

The API runs at `http://127.0.0.1:8000/`, and the interactive documentation is available at `http://127.0.0.1:8000/api/docs/`.

## Project structure

```
task_management_project/
  task_management_app/
    models.py            Task model (title, status, priority, due date, owner)
    serializers.py        RegisterSerializer, TaskSerializer
    views.py              RegisterView, TaskListCreateView, TaskDetailView
    urls.py                App-level routes (auth + tasks)
    admin.py                Task registered in Django admin
  task_management_project/
    settings.py            REST_FRAMEWORK, SIMPLE_JWT, SPECTACULAR_SETTINGS, MySQL config
    urls.py                  Root routes + Swagger/OpenAPI schema endpoints
  .env                       Secrets and database credentials (not committed to git)
  requirements.txt
```

## Endpoints

| Method | Endpoint                | Description                              | Auth required |
| ------ | ------------------------ | ----------------------------------------- | -------------- |
| POST   | `/api/auth/register/`     | Create a new user account                 | No             |
| POST   | `/api/auth/login/`         | Log in and receive JWT tokens              | No             |
| POST   | `/api/auth/refresh/`        | Exchange a refresh token for a new access token | No      |
| GET    | `/api/tasks/`                | List the logged-in user's tasks (filter/search/order supported) | Yes |
| POST   | `/api/tasks/`                  | Create a new task                         | Yes            |
| GET    | `/api/tasks/{id}/`                | Retrieve a single task                    | Yes            |
| PUT    | `/api/tasks/{id}/`                  | Replace a task's full data                | Yes            |
| PATCH  | `/api/tasks/{id}/`                    | Partially update a task                   | Yes            |
| DELETE | `/api/tasks/{id}/`                      | Delete a task                             | Yes            |
| GET    | `/api/docs/`                              | Swagger UI — browse and test every endpoint | No         |

## Features

- Custom user registration with password confirmation and Django's built-in password strength validation
- JWT-based authentication with access/refresh token rotation
- Per-user task isolation — every query is scoped to `request.user`, enforced at the queryset level
- Filtering by `status` and `priority`, full-text search across `title` and `description`, and ordering by `due_date`, `priority`, or `created_at`
- Paginated task lists (10 per page by default)
- Auto-generated, interactive API documentation via Swagger UI

## Environment variables

| Variable       | Description                                  |
| --------------- | --------------------------------------------- |
| `SECRET_KEY`      | Django's cryptographic secret key               |
| `DEBUG`             | `True` for local development, `False` in production |
| `DB_NAME`              | MySQL database name                            |
| `DB_USER`               | MySQL username                                  |
| `DB_PASSWORD`             | MySQL password                                  |
| `DB_HOST`                   | MySQL host (e.g. `localhost`)                   |
| `DB_PORT`                     | MySQL port (default `3306`)                     |

## Known limitations

- There is no email verification step after registration — an account is active as soon as it is created.
- Task ownership is transferred at creation time only; there is no endpoint to reassign a task to another user.
- The project does not yet include automated tests.

## License

This project is for personal/educational and portfolio use.

## Live Demo

*(Add your deployed PythonAnywhere link here once the project is live, e.g. `https://yourusername.pythonanywhere.com/api/docs/`)*
