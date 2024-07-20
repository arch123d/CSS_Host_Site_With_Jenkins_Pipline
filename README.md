```
pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'shan20000/css_template'
                }
    stages {
        stage('Pull Source Code') {
            steps {
                git 'https://github.com/Shantanu20000/CSS_Host_Site_With_Jenkins_Pipline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker Image') {
            environment {
                registryCredential = 'dockerhub-credentials-id'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        docker.image("${DOCKERHUB_REPO}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
         stage('Deploy to Kubernetes') {
            environment {
                AWS_CREDENTIALS = 'awscred'
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS]]) {
                    script {
                        sh """
                    aws eks update-kubeconfig --name my-cluster --region ap-south-1 --kubeconfig /tmp/config
                    kubectl apply -f k8s_onepod.yaml  --kubeconfig=/tmp/config 
                    """
                    }
                }
            }
        }
    }
}
```
# To Deploy latest Image with Build Count
```
pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = 'shan20000/css_template'
                }
    stages {
        stage('Pull Source Code') {
            steps {
                git 'https://github.com/Shantanu20000/CSS_Host_Site_With_Jenkins_Pipline.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKERHUB_REPO}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push Docker Image') {
            environment {
                registryCredential = 'dockerhub-credentials-id'
            }
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', registryCredential) {
                        docker.image("${DOCKERHUB_REPO}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
         stage('Deploy to Kubernetes') {
            environment {
                AWS_CREDENTIALS = 'awscred'
            }
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: AWS_CREDENTIALS]]) {
                    script {
                        sh """
                    aws eks update-kubeconfig --name my-cluster --region ap-south-1 --kubeconfig /tmp/config
                    kubectl set image deployment/css-deployment css=shan20000/css_template:${env.BUILD_NUMBER}  --kubeconfig=/tmp/config
                    """
                    }
                }
            }
        }
    }
}
```
