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

        stage('Build Docker Image') {
            steps {
                script {
                    // Switch to BuildKit if needed
                    sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Scan Docker Image with Trivy') {
            steps {
                sh '''
                if ! command -v trivy &> /dev/null
                then
                    echo "Installing Trivy..."
                    curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin
                fi

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
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy to AWS EC2 (Optional)') {
            when {
                expression { return env.EC2_IP && env.SSH_KEY }
            }
            steps {
                sshagent (credentials: ['ec2-ssh-key']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$EC2_IP << EOF
                    docker pull $IMAGE_NAME:$IMAGE_TAG
                    docker stop dotnetapp || true
                    docker rm dotnetapp || true
                    docker run -d --name dotnetapp -p 80:80 $IMAGE_NAME:$IMAGE_TAG
                    EOF
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Pipeline completed."
        }
    }
}
