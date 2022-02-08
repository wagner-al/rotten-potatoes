pipeline {
    agent any

    stages {
        stage('Checkout Source') {
            steps {
                git url:'https://github.com/fabricioveronez/rotten-potatoes.git', branch:'main'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    dockerapp = docker.build("fabricioveronez/rotten-potatoes:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                        dockerapp.push('latest')
                        dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            agent {
                kubernetes {
                    cloud 'kubernetes'
                }
            }
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                script {
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/deployment.yaml'
                    sh 'cat ./k8s/deployment.yaml'
                    kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
                }
            }
        }
    }

    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
