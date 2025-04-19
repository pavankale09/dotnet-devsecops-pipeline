pipeline {
    agent any

    environment {
        IMAGE_NAME = "kpavan09/devsecops-dotnet-app"
        IMAGE_TAG = "latest"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git 'https://github.com/pavankale09/dotnet-devsecops-pipeline.git'
            }
        }

        stage('Build .NET Application') {
            steps {
                sh 'dotnet restore'
                sh 'dotnet build --configuration Release'
                sh 'dotnet publish -c Release -o out'
            }
        }

        stage('Check Dockerfile Exists') {
            steps {
                script {
                    if (!fileExists('Dockerfile')) {
                        error "❌ Dockerfile not found! Please add a Dockerfile at the root of your repository."
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Scan Docker Image with Trivy') {
            steps {
                sh '''
                if ! command -v trivy &> /dev/null
                then
                    echo "🔧 Installing Trivy..."
                    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
                fi

                echo "🔍 Scanning Docker image with Trivy..."
                trivy image --severity HIGH,CRITICAL --exit-code 0 --format table $IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }

        stage('Push Docker Image (Optional)') {
            when {
                expression { return env.DOCKER_USER && env.DOCKER_PASS }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "🔐 Logging in to Docker Hub..."
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        
