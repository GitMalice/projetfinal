pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/GitMalice/projetfinal.git'
            }
        }
        
        stage('Test') {
            steps {
                script {
                    echo "Vérification de la présence du fichier test_app.py dans l'image Docker"
                    // Exécuter une commande pour lister le contenu du répertoire /app
                    sh 'docker run --rm dockmalice/flaskapp:latest ls -l /app'

                    echo "Exécution des tests avec pytest"
                    // Exécuter pytest pour le fichier de test test_app.py
                    sh 'docker run --rm dockmalice/flaskapp:latest pytest /app/test_app.py --maxfail=1 --disable-warnings -q'
                }
            }
        } 
        
        stage('Owasp') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DC'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.url=http://localhost:9000/ \
                        -Dsonar.projectName=python-webapp \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=python-webapp'''
                }
            }
        }
        
        stage('Build & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker') {
                        sh 'docker build -t flaskapp:latest -f ./Dockerfile .'
                        sh 'docker tag flaskapp:latest dockmalice/flaskapp:latest'
                        sh 'docker push dockmalice/flaskapp:latest'
                    }
                }
            }
        }
        
        stage('Trigger cd pipeline') {
            steps {
                build job: 'cd_pipeline', wait: true
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
