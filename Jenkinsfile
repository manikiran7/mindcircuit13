pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    environment {
        SONARQUBE_ENV = 'mysonarqube'
        DOCKER_IMAGE = 'manikiran7/firstrepo'
        DOCKER_CREDENTIAL = 'dockerhub-creds'
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/manikiran7/mindcircuit13.git'
            }
        }
        stage('Code Qulaity Check') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    sh 'mvn clean verify sonar:sonar'
                }
            }
        }
        stage('Maven Builds') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Docker Build & Push image') {
            steps {
                script {
                    def tag = "${BUILD_NUMBER}"
                    withCredentials([usernamePassword(credentialsId: "${DOCKER_CREDENTIAL}", usernameVariable: 'DOCKERUSER', passwordVariable: 'DOCKERPASS')]) {
                        sh """
                        docker images | grep ${DOCKER_IMAGE} | awk '{print \$3}' | xargs -r docker rmi -f || true
                        docker build -t ${DOCKER_IMAGE}:${tag} .
                        echo \$DOCKERPASS | docker login -u \$DOCKERUSER --password-stdin
                        docker push ${DOCKER_IMAGE}:${tag}
                        """
                    }
                }
            }
        }
        stage('Update GitOps Deployment File') {
            steps {
                script {
                    def tag = "${BUILD_NUMBER}"
                    def image = "${DOCKER_IMAGE}:${tag}"
                    withCredentials([usernamePassword(credentialsId: 'gitops-creds', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh """
                        rm -rf mindcircuit13
                        git clone https://${GIT_USER}:${GIT_PASS}@github.com/manikiran7/mindcircuit13.git
                        cd mindcircuit13
                        sed -i 's|image:.*|image: ${image}|' k8s/deployment.yaml
                        git config user.email "jenkins@example.com"
                        git config user.name "jenkins"
                        git add k8s/deployment.yaml
                        git commit -m "Update image to ${image}"
                        git push origin main
                        """
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
