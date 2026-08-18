pipeline {
    agent any

    triggers {
        // Requirement 1: Scheduled release on 25th of every month
        cron('0 0 25 * *')
        // Requirement 2: CodeBuild/Pipeline triggered on master commit
        pollSCM('H/5 * * * *')
    }

    environment {
        DOCKER_IMAGE = "thiyagup/analytics-web"
        BUILD_TAG    = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/thiyagup/website.git'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                        sh "docker build -t ${DOCKER_IMAGE}:${BUILD_TAG} -t ${DOCKER_IMAGE}:latest ."
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_TAG}"
                        sh "docker push ${DOCKER_IMAGE}:latest"
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                withCredentials([file(credentialsId: 'k8s-kubeconfig', variable: 'KUBECONFIG')]) {
                    sh "kubectl --kubeconfig=\$KUBECONFIG apply -f k8s-deployment.yml"
                    sh "kubectl --kubeconfig=\$KUBECONFIG rollout restart deployment/analytics-website"
                }
            }
        }
    }
}
