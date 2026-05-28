pipeline {
    agent any

    stages {

        stage('Checkout from GitHub') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Projects-for-Practice-Org/spring-boot-mongo-docker-kkfunda.git'
            }
        }

        stage('Setup KubeConfig') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-eks-cd'
                ]]) {

                    sh '''
                        aws eks update-kubeconfig \
                        --region us-east-1 \
                        --name my-demo-cluster
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-eks-cd'
                ]]) {

                    sh '''
                        kubectl apply -f springBootMongo.yaml
                    '''
                }
            }
        }

        stage('Wait for Pods to be Ready') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-eks-cd'
                ]]) {

                    sh '''
                        kubectl rollout status deployment/springappdeployment --timeout=300s

                        kubectl rollout status deployment/mongodbdeployment --timeout=300s
                    '''
                }
            }
        }

        stage('Verify Pods and Services') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-eks-cd'
                ]]) {

                    sh '''
                        kubectl get pods -o wide
                        kubectl get svc
                    '''
                }
            }
        }
    }
}
