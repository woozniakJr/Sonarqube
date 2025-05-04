pipeline {
    agent none // Pas d'agent global, on en spécifie un par stage

    environment {
        DOCKER_CREDENTIALS_ID = 'dimanche' // ID des credentials Docker Hub
        DOCKERHUB_USER = 'mouhamed2555' // Ton nom d'utilisateur Docker Hub
        SONARQUBE_URL = 'http://localhost:9000/' // URL de ton serveur SonarQube
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
                    withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
                        withSonarQubeEnv('SonarQube') {
                            sh "${tool 'SonarQube-Scanner'}/bin/sonar-scanner -Dsonar.token=$SONAR_TOKEN -Dsonar.host.url=$SONARQUBE_URL"
                        }
                    }
                }
            }
        }

        stage('SonarQube Analysis for Frontend') {
            agent any
            steps {
                dir('Frontend') {
                    echo '🔍 Analyse SonarQube du Frontend...'
                    withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
                        withSonarQubeEnv('SonarQube') {
                            sh "${tool 'SonarQube-Scanner'}/bin/sonar-scanner -Dsonar.token=$SONAR_TOKEN -Dsonar.host.url=$SONARQUBE_URL"
                        }
                    }
                }
            }
        }

        stage('Build & Test Django app') {
            agent {
                docker {
                    image 'python:3.12-slim'
                    args '-u root:root'
                }
            }
            steps {
                dir('Backend/odc') {
                    echo "⚙️ Création de l'environnement virtuel et tests Django"
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        python3 manage.py test
                    '''
                }
            }
        }

        stage('Build & Test React app') {
            agent {
                docker {
                    image 'node:20-alpine' // Node.js stable version
                    args '-u root:root'
                }
            }
            steps {
                dir('Frontend') {
                    echo "⚙️ Installation des dépendances et build React"
                    sh '''
                        npm install
                        npm run build
                    '''
                }
            }
        }

        stage('Build Docker images') {
            agent any
            steps {
                echo '🐳 Construction des images Docker...'

                dir('Backend/odc') {
                    sh "docker build -t ${DOCKERHUB_USER}/userprofile_backend:latest -f Dockerfile ."
                }

                dir('Frontend') {
                    sh "docker build -t ${DOCKERHUB_USER}/userprofile_frontend:latest -f Dockerfile ."
                }
            }
        }

        stage('Push Docker images') {
            agent any
            steps {
                echo "🚀 Envoi des images Docker sur Docker Hub..."
                withCredentials([usernamePassword(credentialsId: 'userprofile-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login -u "$DOCKERHUB_USER" --password-stdin
                        docker push $DOCKERHUB_USER/userprofile_backend:latest
                        docker push $DOCKERHUB_USER/userprofile_frontend:latest
                    '''
                }
                echo "✅ Images Docker envoyées avec succès"
            }
        }

        stage('Run the App') {
            agent any
            steps {
                echo '🚀 Lancement de l\'application avec Docker Compose...'
                sh '''
                    docker-compose down || true
                    docker-compose up --build -d
                '''
            }
        }
    }

    post {
        always {
            echo '🧹 Nettoyage...'
            cleanWs()
            echo '✅ Pipeline terminé avec succès.'
        }
    }
}
