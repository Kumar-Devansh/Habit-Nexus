# HabitNexus

HabitNexus is a Flask habit tracking application for students and developers. It supports daily routines, progress analytics, DSA tracking, friend features, developer notifications, and PostgreSQL-backed production deployment.

## Current Deployment Stack

| Layer | Tool |
| --- | --- |
| Backend | Flask |
| App server | Gunicorn |
| Database | PostgreSQL |
| Container runtime | Docker |
| Local orchestration | Docker Compose |
| Reverse proxy | Nginx |
| CI/CD | Jenkins |
| Image registry | DockerHub |
| Image security scan | Trivy |
| Server | AWS EC2 Ubuntu |

## Architecture

```text
Internet
  -> Nginx :80/:443
  -> Docker published port :8000
  -> Gunicorn + Flask web container
  -> PostgreSQL container
```

Jenkins builds the app image, runs a Python compile smoke check, scans the image with Trivy, pushes the image to DockerHub, and deploys the app with Docker Compose.

## Project Structure

```text
Habit_Nexus/
  app.py
  database.py
  requirements.txt
  Dockerfile
  docker-compose.yml
  Jenkinsfile
  DOCKER_JENKINS.md
  PRODUCTION.md
  templates/
  static/
  scripts/
```

## Environment Variables

Create `.env` on the EC2 server. Do not commit real secrets.

```env
APP_ENV=production
SECRET_KEY=replace-with-a-long-random-secret
SESSION_COOKIE_SECURE=0
DEVELOPER_EMAILS=your-admin-email@example.com

POSTGRES_DB=habitnexus
POSTGRES_USER=habitnexus_user
POSTGRES_PASSWORD=replace-with-a-strong-db-password
WEB_PORT=8000

DOCKER_IMAGE=devansh0111/habitnexus
DOCKER_TAG=latest

DB_POOL_MIN=1
DB_POOL_MAX=5
DB_CONNECT_TIMEOUT=5
DB_KEEPALIVES=1
DB_KEEPALIVES_IDLE=30
DB_KEEPALIVES_INTERVAL=10
DB_KEEPALIVES_COUNT=5
DB_POOL_HEALTH_CHECK_ATTEMPTS=2
```

Keep `SESSION_COOKIE_SECURE=0` while testing with `http://YOUR_EC2_IP`. Change it to `1` only after HTTPS is configured.

## Run With Docker Compose

Build and start:

```bash
docker compose build
docker compose up -d
```

Initialize database tables after the first start:

```bash
docker compose exec web python -c "from database import init_db; init_db()"
```

Check status and logs:

```bash
docker compose ps
docker compose logs -f web
docker compose logs -f db
```

Test locally on EC2:

```bash
curl http://127.0.0.1:8000
```

## Nginx Reverse Proxy

Use Nginx on port `80` and proxy to the Docker app port:

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_PUBLIC_IP;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

## Jenkins CI/CD

The pipeline is defined in `Jenkinsfile`.

Required Jenkins credentials:

| ID | Kind | Purpose |
| --- | --- | --- |
| `habitnexus-secret-key` | Secret text | Flask `SECRET_KEY` |
| `habitnexus-postgres-password` | Secret text | PostgreSQL password |
| `dockerhub-credentials` | Username with password | DockerHub username and access token |

Pipeline stages:

```text
Workspace: Cleanup
Git: Checkout Source
Config: Prepare Runtime Environment
Docker: Build Image
Verify: Python Compile Check
Security: Trivy Image Scan
DockerHub: Push Image
Deploy: Docker Compose
```

Default DockerHub image:

```text
devansh0111/habitnexus
```

The pipeline creates tags like:

```text
BUILD_NUMBER-shortGitCommit
latest
```

Example:

```text
devansh0111/habitnexus:18-a1b2c3d
devansh0111/habitnexus:latest
```

Jenkins parameters:

| Parameter | Default | Description |
| --- | --- | --- |
| `DOCKER_IMAGE` | `devansh0111/habitnexus` | DockerHub image name |
| `DOCKER_TAG` | blank | Optional manual tag |
| `PUSH_LATEST` | true | Also push `latest` |
| `DEPLOY_APP` | true | Deploy after build |
| `RUN_TRIVY_SCAN` | true | Run Trivy before push/deploy |
| `TRIVY_SEVERITY` | `HIGH,CRITICAL` | Vulnerability severities to report |
| `TRIVY_EXIT_CODE` | `1` | Fail build on matching findings |
| `WEB_PORT` | `8000` | Host port for app |
| `EMAIL_TO` | project email | Notification recipient |

For details, see [DOCKER_JENKINS.md](DOCKER_JENKINS.md).

## Trivy

Trivy runs through Docker:

```text
aquasec/trivy:latest
```

No direct Trivy install is required on Jenkins. In production, keep:

```text
TRIVY_EXIT_CODE=1
```

For temporary testing, use:

```text
TRIVY_EXIT_CODE=0
```

This reports vulnerabilities without failing the build.

## Useful Commands

Restart the app:

```bash
docker compose restart web
```

Rebuild and deploy:

```bash
docker compose up -d --build
```

Stop containers without deleting database data:

```bash
docker compose down
```

Check port `8000`:

```bash
sudo ss -ltnp | grep :8000
```

Backup PostgreSQL:

```bash
docker compose exec db pg_dump -U habitnexus_user habitnexus > habitnexus_backup.sql
```

Restore PostgreSQL:

```bash
docker compose exec -T db psql -U habitnexus_user habitnexus < habitnexus_backup.sql
```

## Manual Production Notes

Manual Gunicorn/systemd deployment is documented in [PRODUCTION.md](PRODUCTION.md). The recommended deployment path for this project is now Docker Compose with Jenkins automation.

## Security Notes

- Do not commit `.env`.
- Use Jenkins credentials for secrets.
- Use a DockerHub access token, not your DockerHub account password.
- Keep `SESSION_COOKIE_SECURE=0` only for plain HTTP testing.
- Switch `SESSION_COOKIE_SECURE=1` after HTTPS is active.
- Keep Trivy enabled before pushing/deploying images.

## Author

Kumar Devansh

GitHub: <https://github.com/Kumar-Devansh>

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE).
