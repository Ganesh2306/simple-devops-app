pipeline {
    agent any

    environment {
        // Your Docker Hub image
        DOCKER_IMAGE = "ganeshmore/simple-devops-app:latest"

        // Ansible paths inside the repo
        ANSIBLE_INVENTORY = "ansible/inventory.ini"
        ANSIBLE_PLAYBOOK  = "ansible/deploy.yml"
    }

    stages {

        stage('Checkout') {
            steps {
                // Jenkins (Pipeline from SCM) already checked out the code.
                // We just print the workspace contents for confirmation.
                echo "Source code already checked out by Jenkins (SCM). Listing files:"
                sh 'pwd && ls -R'
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
                withCredentials([
                    usernamePassword(
                        credentialsId: '94d73f9c-5f13-4df9-8347-458fd55aa806',
                           usernameVariable: 'DOCKER_USER' ,
                           passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Pushing Docker image to Docker Hub: ${DOCKER_IMAGE}"
                sh """
                    docker push ${DOCKER_IMAGE}
                """
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo "Deploying with Ansible using image: ${DOCKER_IMAGE}"
                sh """
                    ansible-playbook -i ${ANSIBLE_INVENTORY} ${ANSIBLE_PLAYBOOK} \
                      --extra-vars "docker_image_name=${DOCKER_IMAGE}"
                """
            }
        }
    }
}

