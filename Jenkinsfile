pipeline {
    agent any

    environment {
        SONAR_TOKEN = "squ_620aa3b7c4180ff14a891831c2d78ac639fc5c13"
        DOCKER_IMAGE = "kpavan09/devsecops-dotnet-app:latest"
        DOCKER_USERNAME = "kpavan09"
        DOCKER_PASSWORD = "Pavan@0910" // ⚠️ Replace this or use Jenkins Credentials instead
    }
    
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

        stage('Build Docker Image') {
            steps {
                dir('dotnet-devsecops-pipeline') {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
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

      
        stage('Docker Build & Push') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                sh """
                    docker build -t $DOCKER_IMAGE .
                    echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin
                    docker push $DOCKER_IMAGE
                """
            }
        }

        stage('Deploy to Kubernetes') {
            when {
                expression { currentBuild.currentResult == 'SUCCESS' }
            }
            steps {
                sh 'kubectl apply -f k8s/deployment.yaml'
                sh 'kubectl apply -f k8s/service.yaml'
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed! Please check logs and fix the issue."
        }
        success {
            echo "Pipeline completed successfully 🚀"
        }
    }
}
