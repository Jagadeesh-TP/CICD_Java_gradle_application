pipeline {
    agent any

    stages {
        stage("Sonar Quality Check") {
            steps {
                script {
                    docker.image('openjdk:11').inside {
                        withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube'
                        }
                    }
                }

                script {  // ✅ Wrap the Quality Gate check inside a script block
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}
