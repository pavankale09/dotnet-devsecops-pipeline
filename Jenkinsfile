pipeline {
    agent any

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = "1"
        SONAR_TOKEN = credentials('sonarqube-token') // Placeholder for SonarQube
        DOCKERHUB_USER = credentials('kpavan09')
        DOCKERHUB_PASS = credentials('Pavan@0910')
        IMAGE_NAME = "pavankale09/dotnet-devsecops-app"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/pavankale09/dotnet-devsecops-pipeline.git'
            }
        }

        stage('Restore Dependencies') {
            steps {
                sh 'dotnet restore'
            }
        }

        stage('Build Project') {
            steps {
                sh 'dotnet build --configuration Release'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'dotnet test'
            }
        }

        // SonarQube - Placeholder for now
        stage('Code Quality - SonarQube') {
            steps {
                echo "SonarQube Scan (Will be set up later)"
                // Placeholder logic will go here later
            }
        }

        // Trivy Scan - Placeholder for now
        stage('Security Scan - Trivy') {
            steps {
                echo "Trivy Scan (Will be set up later)"
                // Placeholder logic will go here later
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('', 'dockerhub-credentials') {
                        sh '''
                            docker build -t $IMAGE_NAME .
                            echo $DOCKERHUB_PASS | docker login -u $DOCKERHUB_USER --password-stdin
                            docker push $IMAGE_NAME
                        '''
                    }
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@<EC2_PUBLIC_IP> '
                        docker pull $IMAGE_NAME &&
                        docker stop myapp || true &&
                        docker rm myapp || true &&
                        docker run -d --name myapp -p 80:80 $IMAGE_NAME
                    '
                '''
            }
        }
    }
}
