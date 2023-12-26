pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                script {
                    dockerImage = docker.build("rabiazaib/rabiazaib-portfolio:${env.BUILD_ID}")
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'docker-cred-r') {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Test') {
            steps {
                sh 'ls -l index.html' // Simple check for index.html
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Deploy the new version
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "Aqsa", 
                                transfers: [sshTransfer(
                                    execCommand: """
                                        docker pull rabiazaib/rabiazaib-portfolio:${env.BUILD_ID}
                                        docker stop rabiazaib-portfolio-container || true
                                        docker rm rabiazaib-portfolio-container || true
                                        docker run -d --name rabiazaib-portfolio-container -p 401:80 rabiazaib/rabiazaib-portfolio:${env.BUILD_ID}
                                    """
                                )]
                            )
                        ]
                    )
                }
            }
        }
    }

    post {
        failure {
            mail(
                to: 'sp20-bcs-037@cuiatk.edu.pk',
                subject: "Failed Pipeline: ${env.JOB_NAME} [${env.BUILD_NUMBER}]",
                body: "Something is wrong with the build ${env.BUILD_URL}"
            )
        }
    }
}
