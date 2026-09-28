pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'YOUR_DOCKERHUB_USERNAME'   // <-- change me
        BACKEND_IMAGE  = "${DOCKERHUB_USER}/gestion-projets-backend"
        FRONTEND_IMAGE = "${DOCKERHUB_USER}/gestion-projets-frontend"
        IMAGE_TAG      = "${BUILD_NUMBER}"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Test Backend') {
            steps {
                dir('backend') {
                    sh 'mvn -B test'
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: 'backend/target/surefire-reports/*.xml'
                }
            }
        }

        stage('Docker Build') {
            parallel {
                stage('Backend image') {
                    steps {
                        sh '''
                            docker build -t $BACKEND_IMAGE:$IMAGE_TAG -t $BACKEND_IMAGE:latest ./backend
                        '''
                    }
                }
                stage('Frontend image') {
                    steps {
                        sh '''
                            docker build -t $FRONTEND_IMAGE:$IMAGE_TAG -t $FRONTEND_IMAGE:latest ./frontend
                        '''
                    }
                }
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker push $BACKEND_IMAGE:$IMAGE_TAG
                        docker push $BACKEND_IMAGE:latest
                        docker push $FRONTEND_IMAGE:$IMAGE_TAG
                        docker push $FRONTEND_IMAGE:latest
                    '''
                }
            }
        }

        stage('Deploy (Docker Compose)') {
            steps {
                sh '''
                    docker compose down || true
                    docker compose up -d
                    docker compose ps
                '''
            }
        }
    }

    post {
        always {
            sh 'docker logout || true'
        }
        success {
            echo "Delivered ${BACKEND_IMAGE}:${IMAGE_TAG} and ${FRONTEND_IMAGE}:${IMAGE_TAG}"
        }
        failure {
            echo 'Pipeline failed, check the stage logs.'
        }
    }
}
