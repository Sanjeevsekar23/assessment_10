
pipeline {
    agent any

    parameters {
        string(
            name: 'IMAGE_TAG',
            defaultValue: 'v1',
            description: 'Docker image tag to build and deploy'
        )
    }

    environment {
        IMAGE = 'YOUR_DOCKERHUB_USERNAME/smart-campus-energy'
        DOCKER = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Users\\Admin\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        KUBECONFIG = 'C:\\Users\\Admin\\.kube\\config'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Sanjeevsekar23/assessment_10.git'
            }
        }

        stage('Parallel Tests') {
            parallel {
                stage('HTML Check') {
                    steps {
                        bat 'findstr /C:"Smart Campus Energy Monitoring" index.html'
                    }
                }
                stage('Dockerfile Check') {
                    steps {
                        bat 'if exist Dockerfile (echo Dockerfile found) else (exit /b 1)'
                    }
                }
            }
        }

        stage('Build Image') {
            steps {
                bat '"%DOCKER%" build -t %IMAGE%:%IMAGE_TAG% .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USERNAME',
                    passwordVariable: 'DOCKER_PASSWORD'
                )]) {
                    bat 'echo %DOCKER_PASSWORD%| "%DOCKER%" login -u %DOCKER_USERNAME% --password-stdin'
                    bat '"%DOCKER%" push %IMAGE%:%IMAGE_TAG%'
                }
            }
        }

        stage('Deploy') {
            steps {
                bat 'set "KUBECONFIG=%KUBECONFIG%" && "%KUBECTL%" set image deployment/campus-energy campus-energy=%IMAGE%:%IMAGE_TAG%'
                bat 'set "KUBECONFIG=%KUBECONFIG%" && "%KUBECTL%" rollout status deployment/campus-energy --timeout=180s'
            }
        }

        stage('Verify') {
            steps {
                bat 'set "KUBECONFIG=%KUBECONFIG%" && "%KUBECTL%" get deployments,pods,services'
            }
        }
    }

    post {
        always {
            bat '"%DOCKER%" logout'
        }
        success {
            echo 'Campus energy application deployed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
