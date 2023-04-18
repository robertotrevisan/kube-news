pipeline {
    agent any

    stages {
        stage ('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("fabricioveronez/kube-news:${env.BUILD_ID}", '-f ./src/Dockerfile ./src') 
                }                
            }
        }

        stage ('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes Digital Ocean') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f ./k8s/deployment.yaml'                    
                }
            }
        }
        
        stage('Deploy Kubernetes EKS') {
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                withAWS(credentials:'awskey') {
                    sh 'aws eks update-kubeconfig --name jornada-eks --region us-east-1'
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'kubectl apply -f ./k8s/deployment.yaml'                    
                }
            }
        }
    }
}
