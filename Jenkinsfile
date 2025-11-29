pipeline {
    agent any

    environment {
        // Docker image name (change this to your Docker Hub username)
        DOCKER_IMAGE = "ganeshmore/simple-devops-app:latest"

        // Ansible files
        ANSIBLE_INVENTORY = "ansible/inventory.ini"
        ANSIBLE_PLAYBOOK = "ansible/deploy.yml"
    }

    stage('Checkout') {
      steps {
            echo "Code already checked out by Jenkins (Pipeline from SCM). Skipping extra git checkout."
           }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image: ${DOCKER_IMAGE}"
                sh """
                    docker build -t ${DOCKER_IMAGE} .
                """
            }
        }

        stage('Docker Login') {
            steps {
                echo "Logging in to Docker Hub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to Docker Hub..."
                sh """
                    docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo "Running Ansible playbook to deploy to app server..."
                sh """
                    ansible-playbook -i ${ANSIBLE_INVENTORY} ${ANSIBLE_PLAYBOOK} \
                      --extra-vars "docker_image_name=${DOCKER_IMAGE}"
                """
            }
        }
    }
}

