pipeline {
    agent {
        label 'agent-node-label'
    }

    environment {
        APP_NAME = 'yanovych_anastasiia'
        DOCKER_IMAGE_NAME = 'yanovych_anastasiia'
        GOCACHE="/home/jenkins/.cache/go-build/"
    }

    stages {
        stage('Clone Repository') {
            steps {
                // Крок клонування репозиторію1
                checkout scm
            }
        }

        stage('Compile') {
            agent {
                // Використання Docker образу з підтримкою Go версії 1.21.3.
                docker {
                    image 'golang:1.21.3'
                    reuseNode true
                }
            }
            steps {
                // Компіляція проекту на мові Go
                sh "CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -a -ldflags '-w -s -extldflags \"-static\"' -o ${APP_NAME} ."
            }
        }

        stage('Unit Testing') {
            agent {
                // Використання Docker образу з підтримкою Go версії 1.21.3.
                docker {
                    image 'golang:1.21.3'
                    reuseNode true
                }
            }
            steps {
                // Виконання юніт-тестів
                sh 'go test ./...'
            }
        }

        stage('Archive Artifact and Build Docker Image') {
            parallel {
                stage('Archive Artifact') {
                    steps {
                        // Створення TAR-архіву артефакту
                        sh "tar -czf ${APP_NAME}-${BUILD_NUMBER}.tar.gz ${APP_NAME}"
                    }
                }

                stage('Build Docker Image') {
                    steps {
                        // Створення Docker образу
                        script {
                            // Створення Dockerfile
                            writeFile file: 'Dockerfile', text: "FROM scratch\nCOPY ${APP_NAME} /${APP_NAME}\nCMD [\"/${APP_NAME}\"]"

                            // Збірка Docker образу з аргументом APP_NAME
                            sh "docker build -t ${DOCKER_IMAGE_NAME}:${BUILD_NUMBER} --build-arg APP_NAME=${APP_NAME} ."
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            // Архівація успішна, артефакт готовий для використання та збереження
            archiveArtifacts artifacts: "${APP_NAME}-${BUILD_NUMBER}.tar.gz", onlyIfSuccessful: true
        }
        always {
            // Завершення пайплайну, можна додати додаткові кроки (наприклад, розгортання) за потребою
            echo 'Pipeline finished.'
        }
    }
}
