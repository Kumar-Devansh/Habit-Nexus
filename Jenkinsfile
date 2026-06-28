pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
        timestamps()
        disableConcurrentBuilds()
    }

    triggers {
        pollSCM('H/2 * * * *')
    }

    parameters {
        string(
            name: 'DOCKER_IMAGE',
            defaultValue: 'devansh0111/habitnexus',
            description: 'DockerHub image name, for example: dockerhub-username/habitnexus'
        )
        string(
            name: 'DOCKER_TAG',
            defaultValue: '',
            description: 'Optional Docker tag. Leave blank to use the Jenkins build number.'
        )
        booleanParam(
            name: 'PUSH_LATEST',
            defaultValue: true,
            description: 'Also push the Docker image as latest.'
        )
        booleanParam(
            name: 'DEPLOY_APP',
            defaultValue: true,
            description: 'Deploy with Docker Compose after a successful build.'
        )
        string(
            name: 'WEB_PORT',
            defaultValue: '8000',
            description: 'Host port for the Flask/Gunicorn container.'
        )
        string(
            name: 'EMAIL_TO',
            defaultValue: 'devanshup1312@gmail.com',
            description: 'Email address for build notifications.'
        )
    }

    environment {
        COMPOSE_PROJECT_NAME = 'habitnexus'
        DOCKERHUB_CREDENTIALS_ID = 'dockerhub-credentials'
        SECRET_KEY_CREDENTIALS_ID = 'habitnexus-secret-key'
        POSTGRES_PASSWORD_CREDENTIALS_ID = 'habitnexus-postgres-password'
    }

    stages {
        stage('Workspace Cleanup') {
            steps {
                deleteDir()
            }
        }

        stage('Git: Code Checkout') {
            steps {
                checkout scm
                script {
                    env.GIT_COMMIT_SHORT = sh(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()
                    env.IMAGE_TAG = params.DOCKER_TAG?.trim()
                        ? params.DOCKER_TAG.trim()
                        : "${env.BUILD_NUMBER}-${env.GIT_COMMIT_SHORT}"
                    echo "Git commit: ${env.GIT_COMMIT_SHORT}"
                    echo "Docker image: ${params.DOCKER_IMAGE}:${env.IMAGE_TAG}"
                }
            }
        }

        stage('Prepare: Runtime Environment') {
            steps {
                withCredentials([
                    string(credentialsId: env.SECRET_KEY_CREDENTIALS_ID, variable: 'SECRET_KEY'),
                    string(credentialsId: env.POSTGRES_PASSWORD_CREDENTIALS_ID, variable: 'POSTGRES_PASSWORD')
                ]) {
                    sh '''
                        set -eu
                        cat > .env <<EOF
APP_ENV=production
SECRET_KEY=${SECRET_KEY}
SESSION_COOKIE_SECURE=${SESSION_COOKIE_SECURE:-0}
DEVELOPER_EMAILS=${DEVELOPER_EMAILS:-}
POSTGRES_DB=${POSTGRES_DB:-habitnexus}
POSTGRES_USER=${POSTGRES_USER:-habitnexus_user}
POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
WEB_PORT=${WEB_PORT}
DOCKER_IMAGE=${DOCKER_IMAGE}
DOCKER_TAG=${IMAGE_TAG}
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

        stage('Docker: Build Image') {
            steps {
                sh '''
                    set -eu
                    docker compose build web
                    docker image inspect "${DOCKER_IMAGE}:${IMAGE_TAG}" >/dev/null
                '''
            }
        }

        stage('Verify: Static Smoke Check') {
            steps {
                sh '''
                    set -eu
                    docker compose run --rm --no-deps web python -m py_compile app.py database.py
                '''
            }
        }

        stage('DockerHub: Push Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: env.DOCKERHUB_CREDENTIALS_ID,
                        usernameVariable: 'DOCKERHUB_USERNAME',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        set -eu
                        echo "${DOCKERHUB_TOKEN}" | docker login -u "${DOCKERHUB_USERNAME}" --password-stdin
                        docker push "${DOCKER_IMAGE}:${IMAGE_TAG}"
                        if [ "${PUSH_LATEST}" = "true" ]; then
                            docker tag "${DOCKER_IMAGE}:${IMAGE_TAG}" "${DOCKER_IMAGE}:latest"
                            docker push "${DOCKER_IMAGE}:latest"
                        fi
                        docker logout
                    '''
                }
            }
        }

        stage('Deploy: Docker Compose') {
            when {
                expression { return params.DEPLOY_APP }
            }
            steps {
                sh '''
                    set -eu
                    docker compose up -d --remove-orphans
                    docker compose exec -T web python -c "from database import init_db; init_db()"
                    docker compose ps
                '''
            }
        }
    }

    post {
        success {
            script {
                sendBuildEmail(
                    'HabitNexus deployment succeeded',
                    '#90EE90',
                    "Docker image: ${params.DOCKER_IMAGE}:${env.IMAGE_TAG}"
                )
            }
        }
        failure {
            script {
                sendBuildEmail(
                    'HabitNexus deployment failed',
                    '#FFA07A',
                    'Check the attached Jenkins console log.'
                )
            }
        }
        always {
            sh 'docker compose ps || true'
        }
    }
}

def sendBuildEmail(String subjectPrefix, String statusColor, String message) {
    try {
        emailext(
            attachLog: true,
            subject: "${subjectPrefix} - ${currentBuild.currentResult}",
            body: """
                <html>
                <body>
                    <div style="background-color: ${statusColor}; padding: 10px; margin-bottom: 10px;">
                        <p style="color: black; font-weight: bold;">Project: ${env.JOB_NAME}</p>
                    </div>
                    <div style="background-color: #90EE90; padding: 10px; margin-bottom: 10px;">
                        <p style="color: black; font-weight: bold;">Build Number: ${env.BUILD_NUMBER}</p>
                    </div>
                    <div style="background-color: #87CEEB; padding: 10px; margin-bottom: 10px;">
                        <p style="color: black; font-weight: bold;">URL: ${env.BUILD_URL}</p>
                    </div>
                    <p>${message}</p>
                </body>
                </html>
            """,
            to: params.EMAIL_TO,
            mimeType: 'text/html'
        )
    } catch (error) {
        echo "Email notification skipped: ${error}"
    }
}
