pipeline {
    agent any
    environment {
        VERSION = "${env.BUILD_ID}"  
    }
    stages {
        stage("Sonar Quality Check") {
            steps {
                script {
                    docker.image('openjdk:11').inside {
                        sh 'ls -lah && pwd'  // Debugging step
                        sh 'chmod +x gradlew && ls -lah gradlew'
                        sh 'file gradlew'   // Check for CRLF issues
                        withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'bash ./gradlew sonarqube'  // Explicitly use bash
                        }
                    }
                }

                script {  
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
        stage("Docker Build and Push") {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh 'docker info'  // Ensure Docker daemon is running
                        sh """
                        docker build -t 34.47.171.243:8083/springapp:${VERSION} .
                        echo "${docker_password}" | docker login -u admin --password-stdin 34.47.171.243:8083
                        docker push 34.47.171.243:8083/springapp:${VERSION}
                        docker rmi 34.47.171.243:8083/springapp:${VERSION}
                        """  
                    }
                }
            }
        }
    }
}
