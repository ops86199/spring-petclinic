pipeline {
    agent any

    environment {
        REGISTRY = "ops86199"
        IMAGE_NAME = "spring-petclinic"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url:'https://github.com/ops86199/spring-petclinic'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Trivy Scan') {
            steps {
                sh 'trivy image $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER || true'
            }
        }

        stage('Push to DockerHub') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-pass', variable: 'DOCKERHUB_PASS')]) {
                    sh """
                    echo $DOCKERHUB_PASS | docker login -u $REGISTRY --password-stdin
                    docker push $REGISTRY/$IMAGE_NAME:$BUILD_NUMBER
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                kubectl set image deployment/spring-petclinic spring-petclinic=$REGISTRY/$IMAGE_NAME:$BUILD_NUMBER
                kubectl rollout status deployment/spring-petclinic
                '''
            }
        }
    }

    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Build or Deployment Failed!'
        }
    }
}
