pipeline{
    agent any 
   
    stages{
        stage("sonar quality check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                            sh 'chmod +x gradlew'
                            sh './gradlew sonarqube -Dsonar.login=6945dd2cc54250e6dee29431e470c41f13004680'
                    }
                }  
            }
        }
    }
}