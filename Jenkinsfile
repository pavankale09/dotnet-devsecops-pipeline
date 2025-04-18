pipeline {
    agent any

    environment {
        IMAGE_NAME = "kpavan09/dotnet-devsecops-pipeline"
        SONARQUBE = "SonarQube"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/pavankale09/dotnet-devsecops-pipeline.git'
            }
        }

        stage('Restore') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                sh 'dotnet build --no-restore'
            }
        }

        stage('Test') {
            steps {
                sh 'dotnet test --no-build --verbosity normal'
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                sh 'trivy fs . || true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE}") {
                    sh 'dotnet sonarscanner begin /k:"devesecops-dotnet" /d:sonar.login=squ_620aa3b7c4180ff14a891831c2d78ac639fc5c13
                    sh 'dotnet build'
                    sh 'dotnet sonarscanner end /d:sonar.login=squ_620aa3b7c4180ff14a891831c2d78ac639fc5c13
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker build -t $IMAGE_NAME .'
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKER_PASSWORD')]) {
                    sh '''
                    echo $DOCKER_PASSWORD | docker login -u your-dockerhub-username --password-stdin
                    docker push $IMAGE_NAME
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f k8s-deployment.yaml'
            }
        }
    }
}
