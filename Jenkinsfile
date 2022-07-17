pipeline{
    agent any 

    environment{
        VERSION = "${env.BUILD_ID}"
    }
   
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
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }  
            }
        }
        stage("Docker Build and Docker Push"){
            steps{
                script{

                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                        docker build -t 20.110.103.51:8083/springapp:${VERSION} .
                        docker login -u admin -p $docker_password 20.110.103.51:8083
                        docker push 20.110.103.51:8083/springapp:${VERSION}
                        docker rmi 20.110.103.51:8083/springapp:${VERSION}
                    '''   
                    } 
                }
            }
        }
        stage("Identifying misconfigurations using Datree in Helm Charts")
        {
            steps{
                script{
                    dir('kubernetes') {
                        withEnv(['DATREE_TOKEN=GJdx2cP2TCDyUY3EhQKgTc']) {
                            sh 'helm datree test myapp/'
                        }               
                    }
                }
            }
        }
    }

    post {
		always {
			mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "ryantravasso@gmail.com";  
		}
	}
}