pipeline {
    agent any
    environment {
        K8S_PORT = 55673
    }
    stages {
        stage('Build') { 
            steps {
                sh 'mvn clean package'
            }
        }      
        stage('Build Image') {
            steps {
                script {
                    image = docker.build("humbertosandmann/discovery:${env.BUILD_ID}", "-f Dockerfile .")
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credential') {
                        image.push("${env.BUILD_ID}")
                        image.push("latest")
                    }
                }
            }
        }
        stage('Deploy on k8s') {
            steps {
                withCredentials([ string(credentialsId: 'minikube-credential', variable: 'api_token') ]) {
                    sh "kubectl --token $api_token --server https://host.docker.internal:${env.K8S_PORT}  --insecure-skip-tls-verify=true apply -f ./k8s/configmap.yaml"
                    sh "kubectl --token $api_token --server https://host.docker.internal:${env.K8S_PORT}  --insecure-skip-tls-verify=true apply -f ./k8s/deployment.yaml"
                    sh "kubectl --token $api_token --server https://host.docker.internal:${env.K8S_PORT}  --insecure-skip-tls-verify=true apply -f ./k8s/service.yaml"
                }
            }
        }
    }
}