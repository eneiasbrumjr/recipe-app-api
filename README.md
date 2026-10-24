# Recipe API

A Django and Django REST Framework API (application programming interface) for
user accounts, recipes, tags, ingredients, and recipe images. PostgreSQL stores
application data; Docker Compose defines the local development environment.

## Status

On 2026-10-24, [CI run 40112570955](https://github.com/eneiasbrumjr/recipe-app-api/actions/runs/40112570955)
passed the image build, all 64 Django tests, and flake8 for PR head
`02fad8d66c5dfe60f07f4e4b11944d777e42b4fe`, using GitHub's simulated merge with
`main`. This evidence applies to that revision; any later commit, including a
documentation update, needs its own successful run.

Deployment, `collectstatic`, and proxy delivery: **NOT RUN**. Broader dependency
and security validation remain pending.

## Implemented in source

- Account creation, token authentication, and current-user profile management.
- Authenticated, user-scoped recipes, tags, and ingredients.
- Recipe filtering and an image-upload endpoint.
- OpenAPI schema and Swagger documentation routes.
- Django tests covering authentication, user isolation, data operations, and uploads.

## Local development

Use Docker with the Docker Compose plugin. Run these commands from the repository
root. The development configuration supplies its own environment values; copying
`.env.sample` is not required for this configuration.

```sh
docker compose -f docker-compose.yml build
docker compose -f docker-compose.yml up
```

The app waits for PostgreSQL, applies migrations, and starts the Django development
server. Open [interactive documentation](http://localhost:8000/api/docs/).

The development Compose file enables debug mode, contains development credentials,
and publishes ports 8000 and 5432 on the host. Use an isolated development
environment and do not expose these services to untrusted networks.

Stop the services while retaining their named volumes:

```sh
docker compose -f docker-compose.yml down
```

## Routes

| Path | Purpose |
| --- | --- |
| `/api/health-check/` | Basic application response; does not probe database readiness |
| `/api/schema` | OpenAPI schema; this route has no trailing slash |
| `/api/docs/` | Swagger documentation |
| `/api/user/create/` | Account creation |
| `/api/user/token/` | Authentication token |
| `/api/user/me/` | Authenticated user's profile |
| `/api/recipe/recipes/` | Recipes |
| `/api/recipe/tags/` | Tags |
| `/api/recipe/ingredients/` | Ingredients |

For protected endpoints, send `Authorization: Token <token>`.

## Tests and lint

After building the development image:

```sh
docker compose -f docker-compose.yml up -d db
docker compose -f docker-compose.yml run --rm app sh -c "python manage.py wait_for_db && python manage.py test"
docker compose -f docker-compose.yml run --rm app sh -c "flake8"
```

[The workflow](.github/workflows/checks.yml) runs on pushes to `main` and on
pull requests targeting `main` (`opened`, `synchronize`, or `reopened`), including
documentation-only updates. It targets `ubuntu-24.04`, uses the
`docker compose` plugin, and pulls public base images without Docker Hub secrets.
Unauthenticated pulls remain subject to Docker Hub rate limits.

## Repository map

| Path | Responsibility |
| --- | --- |
| `app/core/` | Models, database-wait command, and basic health check |
| `app/user/` | Accounts and token authentication |
| `app/recipe/` | Recipes, related data, uploads, and tests |
| `app/app/` | Django settings and root routes |
| `proxy/` | Nginx configuration for the alternative deployment layout |
| `scripts/run.sh` | Database wait, static-file collection, migrations, and uWSGI startup |

## Deployment boundary

`docker-compose-deploy.yml` defines an alternative app, database, and proxy layout.
Its variables are listed in `.env.sample`: `DB_NAME`, `DB_USER`, `DB_PASS`,
`DJANGO_SECRET_KEY`, and `DJANGO_ALLOWED_HOSTS`. Supply real secrets outside Git.

Before using that layout, validate dependency compatibility and security, runtime
configuration, static/media paths, network exposure, backups, and restoration.
