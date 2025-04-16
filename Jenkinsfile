pipeline {
    agent any

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = "1"
        SONAR_TOKEN = credentials('sonarqube-token') // (Add in Jenkins later)
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

        stage('Code Quality - SonarQube') {
            steps {
                withSonarQubeEnv('MySonarQubeServer') {
                    sh '''
                        dotnet sonarscanner begin /k:"dotnet-devsecops" /d:sonar.login=$SONAR_TOKEN
                        dotnet build
                        dotnet sonarscanner end /d:sonar.login=$SONAR_TOKEN
                    '''
                }
            }
        }

        stage('Security Scan - Trivy') {
            steps {
                sh '''
                    trivy fs . > trivy-report.txt
                    cat trivy-report.txt
                '''
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
