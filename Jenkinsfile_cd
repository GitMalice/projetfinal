pipeline {
    agent any

    environment {
        KUBECTL_HOME = '/usr/local/bin/kubectl'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/GitMalice/projetfinal.git'
            }
        }
        

        stage('Configure kubectl') {
            steps {
                script {
                    withKubeConfig(credentialsId: 'kubernetes_cloud', namespace: '', restrictKubeConfigAccess: false) {
                        sh "${KUBECTL_HOME} apply -f deployment.yaml"
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'All steps done.'
        }
        success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }

    }
}
