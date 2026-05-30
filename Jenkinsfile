pipeline {
    agent any

    tools {
        maven 'maven-3.9.10'
    }

    environment {
        SNYK_TOKEN = credentials('snyk-credentials')
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Cloning Repository..."
                git branch: 'main',
                    url: 'https://github.com/Projects-for-Practice-Org/spring-boot-mongo-docker-kkfunda.git'
                echo "Repository cloned successfully."
            }
        }

        stage('Build') {
            steps {
                echo "Building application..."
                sh 'mvn clean package -DskipTests'
                echo "Build completed successfully."
            }
        }

        stage('SonarQube Scan') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=spring-boot-mongo \
                        -Dsonar.projectName="Spring Boot Mongo Project"
                    '''
                }
            }
        }


        stage('Snyk Scan') {
            steps {
                sh '''
                    snyk auth $SNYK_TOKEN
                    snyk test --all-projects || true
                    snyk monitor --all-projects || true
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Building Docker image..."
                sh 'docker build -t appajichowdary/springbootapp:latest .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                echo "Scanning Docker image with Trivy..."
                sh '''
                     trivy image --exit-code 0 --severity HIGH,CRITICAL appajichowdary/springbootapp:latest
                    trivy image --exit-code 1 --severity CRITICAL appajichowdary/springbootapp:latest || true
                 '''
            }
        }

        stage('Push Docker Image') {
    steps {
        script {
            docker.withRegistry('https://index.docker.io/v1/', 'Docker_cre') {
                sh 'docker push appajichowdary/springbootapp:latest'
            }
        }
    }
}
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }

        success {
            echo 'Pipeline executed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
