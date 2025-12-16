pipeline {
    agent any

    tools {
        jdk 'jdk17'
        maven 'Maven 3.9.3'
    }

    stages {
        stage('Check Docker Access') {
            steps {
                sh 'docker ps'
            }
        }

        stage('GIT') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/eyaezdine/StudentsManagement-DevOps.git'
            }
        }

        stage('Clean & Build Project') {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t eyaezdine/student-management:${BUILD_NUMBER} ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub',
                    usernameVariable: 'DOCKERHUB_USERNAME',
                    passwordVariable: 'DOCKERHUB_PASSWORD'
                )]) {
                    sh '''
                        echo $DOCKERHUB_PASSWORD | docker login -u $DOCKERHUB_USERNAME --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh "docker push eyaezdine/student-management:${BUILD_NUMBER}"
            }
        }


        stage('Check Kubernetes Connection') {
    steps {
        script {
            retry(3) {
                timeout(time: 1, unit: 'MINUTES') {
                    sh """
                        echo "Testing Kubernetes connection..."
                        kubectl cluster-info
                        kubectl get nodes
                        echo "✅ Kubernetes connection OK"
                    """
                }
            }
        }
    }
}
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    def imageTag = "eyaezdine/student-management:${BUILD_NUMBER}"

                    echo "Deploying ${imageTag} to Kubernetes..."

                    sh """
                        kubectl set image deployment/spring-app \
                        spring-app=${imageTag} \
                        -n devops \
                        --record
                    """

                    sh "kubectl rollout status deployment/spring-app -n devops --timeout=300s"

                    sh "kubectl get svc spring-service -n devops"
                }
            }
        }
    }
}
