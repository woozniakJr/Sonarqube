environment {
    DOCKER_CREDENTIALS_ID = 'dimanche'
    DOCKERHUB_USER = 'mouhamed2555'
    SONARQUBE_URL = 'http://localhost:9000/'
}

...

stage('SonarQube Analysis for Backend') {
    agent any
    steps {
        dir('Backend/odc') {
            echo 'Analyse SonarQube du Backend...'
            withCredentials([string(credentialsId: 'sonar_token', variable: 'SONAR_TOKEN')]) {
                withSonarQubeEnv('SonarQube') {
                    sh "${tool 'SonarQube-Scanner'}/bin/sonar-scanner -Dsonar.token=$SONAR_TOKEN -Dsonar.host.url=$SONARQUBE_URL"
                }
            }
        }
    }
}

...

stage('Build Docker image') {
    agent any
    steps {
        echo 'Construction des images Docker...'

        dir('Backend/odc') {
            sh "docker build -t ${DOCKERHUB_USER}/userprofile_backend:latest -f Dockerfile ."
        }

        dir('Frontend') {
            sh "docker build -t ${DOCKERHUB_USER}/userprofile_frontend:latest -f Dockerfile ."
        }
    }
}

...

stage('RUN THE APP') {
    agent any
    steps {
        echo 'Lancement de l\'application...'
        sh '''
            docker-compose down || true
            docker-compose up --build -d
        '''
    }
}
