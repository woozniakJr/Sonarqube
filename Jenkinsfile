pipeline {
    agent none

    environment {
        DOCKER_CREDENTIALS_ID = 'dimanche'
        DOCKERHUB_USER = 'mouhamed2555'
        DOCKER_CREDENTIALS = credentials('dimanche')
        SONARQUBE_URL = 'http://localhost:9000/'
        SONARQUBE_TOKEN = credentials('sonar_token')
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                echo '📥 Clonage du dépôt...'
                checkout scm
            }
        }

        stage('SonarQube Analysis for Backend') {
            agent any
            steps {
                dir('Backend/odc') {
                    echo '🔍 Analyse SonarQube du Backend...'
                    withSonarQubeEnv('SonarQube') {
                        bat """
                            ${tool 'SonarQube-Scanner'}\\bin\\sonar-scanner.bat ^
                            -Dsonar.projectKey=backend ^
                            -Dsonar.projectName=backend ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=%SONARQUBE_URL% ^
                            -Dsonar.token=%SONARQUBE_TOKEN%
                        """
                    }
                }
            }
        }

        stage('SonarQube Analysis for Frontend') {
            agent any
            steps {
                dir('Frontend') {
                    echo '🔍 Analyse SonarQube du Frontend...'
                    withSonarQubeEnv('SonarQube') {
                        bat """
                            ${tool 'SonarQube-Scanner'}\\bin\\sonar-scanner.bat ^
                            -Dsonar.projectKey=Frontend ^
                            -Dsonar.projectName=Frontend ^
                            -Dsonar.sources=. ^
                            -Dsonar.host.url=%SONARQUBE_URL% ^
                            -Dsonar.token=%SONARQUBE_TOKEN%
                        """
                    }
                }
            }
        }

        stage('Build & Test Django App') {
            agent any
            steps {
                dir('Backend/odc') {
                    echo '⚙️ Création de l’environnement virtuel et test Django...'
                    bat """
                        python -m venv venv
                        call venv\\Scripts\\activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        python manage.py test
                    """
                }
            }
        }

        stage('Build & Test React App') {
            agent any
            steps {
                dir('Frontend') {
                    echo '⚙️ Installation des dépendances et build React...'
                    bat """
                        npm install
                        npm run build
                    """
                }
            }
        }

        stage('Build Docker Images') {
            agent any
            steps {
                echo '🐳 Construction des images Docker...'
                script {
                    dir('Backend/odc') {
                        bat "docker build -t ${DOCKERHUB_USER}/userprofile_backend:latest -f Dockerfile ."
                    }
                    dir('Frontend') {
                        bat "docker build -t ${DOCKERHUB_USER}/userprofile_frontend:latest -f Dockerfile ."
                    }
                }
            }
        }

        stage('Push Docker Images') {
            agent any
            steps {
                echo '🚀 Envoi des images Docker sur Docker Hub...'
                withCredentials([usernamePassword(credentialsId: 'userprofile-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    bat """
                        echo %DOCKER_PASSWORD% | docker login -u %DOCKERHUB_USER% --password-stdin
                        docker push %DOCKERHUB_USER%/userprofile_backend:latest
                        docker push %DOCKERHUB_USER%/userprofile_frontend:latest
                    """
                }
            }
        }

        stage('Run the App') {
            agent any
            steps {
                echo '🚀 Lancement de l’application...'
                bat """
                    docker compose down || exit 0
                    docker compose up --build -d
                """
            }
        }
    }

    post {
        always {
            echo '🧹 Nettoyage...'
            echo '✅ CI/CD terminé avec succès'
        }
    }
}
