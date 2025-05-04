pipeline {
    agent none // No default agent, we will specify it in each stage

    environment{
        // Define any environment variables here if needed
        DOCKER_CREDENTIALS_ID = 'dimanche' // Replace with your Docker Hub credentials ID
        DOCKERHUB_USER = 'mouhamed2555' // Replace with your Docker Hub username
        DOCKER_CREDENTIALS = credentials('dimanche')  // Replace with your Docker Hub credentials ID
        SONARQUBE_URL = 'http://localhost:9000/' // Replace with your SonarQube URL
        SONARQUBE_TOKEN = credentials('sonar_token') // Replace with your SonarQube token ID
    }

    stages {
        stage('Checkout'){
            agent any
            steps {
                echo 'Clonage du dépot...'
                checkout scm
            }
        }

        stage('SonarQube Analysis for Backend') {
            agent any
            steps {
                dir('Backend/odc') {
                    echo 'Analyse SonarQube du Backend...'
                    withSonarQubeEnv('SonarQube') {
                        sh "${tool 'SonarQube-Scanner'}/bin/sonar-scanner -Dsonar.token=$SONARQUBE_TOKEN -Dsonar.host.url=$SONARQUBE_URL"
                        
                    }
                }
            }
        }

        stage('Sonarqube Analysis for Frontend') {
            agent any
            steps {
                dir('Frontend') {
                    echo 'Analyse SonarQube du Frontend...'
                    withSonarQubeEnv('SonarQube') {
                        sh "${tool 'SonarQube-Scanner'}/bin/sonar-scanner -Dsonar.token=$SONARQUBE_TOKEN -Dsonar.host.url=$SONARQUBE_URL"
                    }
                }
            }
        }

        // TEST STAGE 
        stage('Build & Test Django app') {
            agent{
                docker {
                    image 'python:3.12-slim' // Use a Python 3.11 Docker image
                    args '-u root:root' // Run as root user to avoid permission issues
                }
            }
            steps {
                dir('Backend/odc') {
                    echo "Création de l'environnement virtuel et test de Django"
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
            agent{
                docker {
                    image 'node:23-alpine' // Use a Node.js 23 Docker image
                    args '-u root:root' // Run as root user to avoid permission issues
                }
            }
            steps {
                dir('Frontend') {
                    echo "Installation des dépendances et test de React"
                    sh '''
                        export PATH=$PATH:/var/lib/jenkins/.nvm/versions/node/v22.15.0/bin/
                        npm install
                        npm run build
                    '''
                }
            }
        }
        // STAGE DE DEPLOIEMENT
        stage('Build Docker image') {
            agent any
            steps {
                echo 'Construction de l\'image Docker Backend...'
                dir('Backend/odc') {
                    script() {
                        echo "🐳 Construction de l'image Docker Backend"
                        sh "docker build -t ${DOCKERHUB_USER}/userprofile_backend:latest -f Dockerfile ."
                            
                        echo "🐳 Construction de l'image Docker Frontend"
                        sh "docker build -t ${DOCKERHUB_USER}/userprofile_frontend:latest ."
                    }
                }
            }
        }

        stage('Push docker images') {
        agent any
        steps {
            echo "🚀 Envoi des images Docker sur Docker Hub"
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


        stage('RUN THE APP') {
            agent any
            steps {
                echo 'Lancement de l\'application...'
                script() {
                    echo "🚀 Lancement de l'application"
                    sh '''
                        docker compose down || true # Stop and remove any existing containers
                        docker compose up --build -d # Build and run the containers in detached mode
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            echo '✅ CI/CD terminé avec succès'
            // Add cleanup commands here
        }
    }
}
