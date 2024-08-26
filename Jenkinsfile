pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'khinesss/nodejs-app:latest'  // image name
        AWS_EC2_USER = 'ec2-user'                   //EC2 instance user
        AWS_EC2_HOST = 'ec2-3-27-213-114.ap-southeast-2.compute.amazonaws.com'              //instance IP/hostname
        // SECRET_FILE = credentials('FileKey')      //pem file
    }

    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t ${DOCKER_IMAGE} .'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'DockerHub', url: 'https://index.docker.io/v1/'   ){
                        sh 'docker push ${DOCKER_IMAGE}'
                    }
                }
            }
        }

        stage('Deploy to AWS EC2') {
            steps {
                script {
                    withCredentials([file(credentialsId: 'FileKey', variable: 'secretFile')]) {
                        // SSH into the EC2 instance and deploy the container
                        sh '''
                        ssh -tt -o StrictHostKeyChecking=no -i $secretFile ${AWS_EC2_USER}@${AWS_EC2_HOST} << EOF
                            docker --version
                            sudo docker pull ${DOCKER_IMAGE} 
                            docker --version
                            (sudo docker stop nodejs-app || true)
                            docker --version
                            (sudo docker rm nodejs-app || true) 
                            docker --version
                            sudo docker run -d --name nodejs-app -p 80:80 ${DOCKER_IMAGE}
                            docker --version
                            exit
                            << EOF
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs() // Clean workspace after the build
        }
    }
}
