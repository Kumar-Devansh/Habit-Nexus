pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = 'habitnexus'
        WEB_PORT = '8000'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Environment') {
            steps {
                withCredentials([
                    string(credentialsId: 'habitnexus-secret-key', variable: 'SECRET_KEY'),
                    string(credentialsId: 'habitnexus-postgres-password', variable: 'POSTGRES_PASSWORD')
                ]) {
                    sh '''
                        set -eu
                        cat > .env <<EOF
APP_ENV=production
SECRET_KEY=${SECRET_KEY}
SESSION_COOKIE_SECURE=1
DEVELOPER_EMAILS=${DEVELOPER_EMAILS:-}
POSTGRES_DB=${POSTGRES_DB:-habitnexus}
POSTGRES_USER=${POSTGRES_USER:-habitnexus_user}
POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
WEB_PORT=${WEB_PORT}
DB_POOL_MIN=${DB_POOL_MIN:-1}
DB_POOL_MAX=${DB_POOL_MAX:-5}
DB_CONNECT_TIMEOUT=${DB_CONNECT_TIMEOUT:-5}
DB_KEEPALIVES=${DB_KEEPALIVES:-1}
DB_KEEPALIVES_IDLE=${DB_KEEPALIVES_IDLE:-30}
DB_KEEPALIVES_INTERVAL=${DB_KEEPALIVES_INTERVAL:-10}
DB_KEEPALIVES_COUNT=${DB_KEEPALIVES_COUNT:-5}
DB_POOL_HEALTH_CHECK_ATTEMPTS=${DB_POOL_HEALTH_CHECK_ATTEMPTS:-2}
EOF
                    '''
                }
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker compose build web'
            }
        }

        stage('Static Smoke Check') {
            steps {
                sh 'docker compose run --rm --no-deps web python -m py_compile app.py database.py'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -eu
                    docker compose up -d --remove-orphans
                    docker compose exec -T web python -c "from database import init_db; init_db()"
                '''
            }
        }
    }

    post {
        always {
            sh 'docker compose ps || true'
        }
    }
}
