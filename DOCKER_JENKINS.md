# Docker and Jenkins Deployment

This setup runs HabitNexus on EC2 with Docker Compose:

- `web`: Flask app served by Gunicorn.
- `db`: PostgreSQL 16 with a persistent Docker volume.
- `profile_uploads`: persistent volume for uploaded profile pictures.

## 1. Prepare EC2

Install Docker and the Compose plugin on your EC2 instance:

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-plugin
sudo usermod -aG docker $USER
newgrp docker
```

Clone or pull the project:

```bash
git clone https://github.com/Kumar-Devansh/Habit_Nexus.git
cd Habit_Nexus
```

## 2. Create `.env`

Create `.env` on EC2. Do not commit this file.

```env
APP_ENV=production
SECRET_KEY=replace-with-a-long-random-secret
SESSION_COOKIE_SECURE=0
DEVELOPER_EMAILS=devanshup1312@gmail.com

POSTGRES_DB=habitnexus
POSTGRES_USER=habitnexus_user
POSTGRES_PASSWORD=replace-with-a-strong-db-password
WEB_PORT=8000

DB_POOL_MIN=1
DB_POOL_MAX=5
DB_CONNECT_TIMEOUT=5
DB_KEEPALIVES=1
DB_KEEPALIVES_IDLE=30
DB_KEEPALIVES_INTERVAL=10
DB_KEEPALIVES_COUNT=5
DB_POOL_HEALTH_CHECK_ATTEMPTS=2
```

Keep `SESSION_COOKIE_SECURE=0` while you are opening the site with `http://YOUR_EC2_IP`.
After you add HTTPS/SSL, change it to `SESSION_COOKIE_SECURE=1`.

`docker-compose.yml` builds `DATABASE_URL` automatically from the Postgres values above:

```text
postgresql://POSTGRES_USER:POSTGRES_PASSWORD@db:5432/POSTGRES_DB
```

## 3. Build And Start

```bash
docker compose build
docker compose up -d
```

Check containers:

```bash
docker compose ps
docker compose logs -f web
```

Initialize database tables after the first start:

```bash
docker compose exec web python -c "from database import init_db; init_db()"
```

Open the app through Nginx or directly while testing:

```bash
curl http://127.0.0.1:8000
```

## 4. Nginx Reverse Proxy 

Keep Nginx on port `80` and proxy to the app container port:

```nginx
server {
    listen 80;
    server_name YOUR_DOMAIN_OR_PUBLIC_IP;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
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

## 5. Jenkins Integration

Install these on the Jenkins host:

- Docker
- Docker Compose plugin
- Git
- Jenkins credentials plugin

Trivy runs through the `aquasec/trivy:latest` Docker image, so you do not need to install Trivy directly on the Jenkins host.

Give Jenkins permission to use Docker:

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

Create Jenkins credentials:

- `habitnexus-secret-key`: Secret text containing your Flask `SECRET_KEY`.
- `habitnexus-postgres-password`: Secret text containing your Postgres password.
- `dockerhub-credentials`: Username with password. Use your DockerHub username and a DockerHub access token.

Create a Pipeline job:

1. Choose `Pipeline script from SCM`.
2. Select Git.
3. Enter your repository URL.
4. Set branch, for example `main`.
5. Script path: `Jenkinsfile`.


// Till here we are done 

The pipeline will:

1. Checkout code.
2. Write `.env` from Jenkins credentials.
3. Build a Docker image.
4. Run a Python compile smoke check.
5. Scan the Docker image with Trivy.
6. Push the image to DockerHub.
7. Start/update containers with `docker compose up -d`.
8. Run `init_db()` inside the web container.

Default image:

```text
devansh0111/habitnexus
```

Change the `DOCKER_IMAGE` Jenkins parameter if your DockerHub repository uses a different name.

The pipeline creates a build tag automatically:

```text
BUILD_NUMBER-shortGitCommit
```

Example:

```text
devansh0111/habitnexus:18-a1b2c3d
devansh0111/habitnexus:latest
```

## 6. Jenkins Parameters

Each build has these parameters:

- `DOCKER_IMAGE`: DockerHub image name, for example `devansh0111/habitnexus`.
- `DOCKER_TAG`: optional manual tag. Leave blank to use `BUILD_NUMBER-shortGitCommit`.
- `PUSH_LATEST`: also push `latest`.
- `DEPLOY_APP`: deploy with Docker Compose after pushing.
- `RUN_TRIVY_SCAN`: scan the image before pushing.
- `TRIVY_SEVERITY`: default `HIGH,CRITICAL`.
- `TRIVY_EXIT_CODE`: default `1`, which fails the build when matching vulnerabilities are found. Use `0` to report only.
- `WEB_PORT`: host port, usually `8000`.
- `EMAIL_TO`: email for Jenkins notifications.

If Trivy fails the build, check the Jenkins console output. For temporary learning/testing, set `TRIVY_EXIT_CODE=0`; for real production, keep it as `1`.

## 7. Common Commands

Restart:

```bash
docker compose restart web
```

Rebuild after code changes:

```bash
docker compose up -d --build
```

View logs:

```bash
docker compose logs -f web
docker compose logs -f db
```

Backup Postgres:

```bash
docker compose exec db pg_dump -U habitnexus_user habitnexus > habitnexus_backup.sql
```

Restore Postgres:

```bash
docker compose exec -T db psql -U habitnexus_user habitnexus < habitnexus_backup.sql
```
