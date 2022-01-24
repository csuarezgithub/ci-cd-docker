pipeline {

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry = "csuarezdocker/vproappdock"
        registryCredential = 'dockerhub'
    }

    stages{

        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
                //sh 'echo test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
                //sh 'echo integrationtest'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
            steps {
               sh 'echo codeanalisys'
            }
        }

        stage('Build App Image'){
            steps {
                script {
                    withDockerServer('tcp://172.17.0.1:2375'){
                        dockerImage = docker.build registry + ":V$BUILD_NUMBER"
                    }
                    
                }
            }
        }

        stage('Upload Image'){
            steps {
                script{
                    docker.withRegistry('', registryCredential){
                        dockerImage.push("V$BUILD_NUMBER")
                        dockerImage.push('latest')
                    }
                }
            }

        }

        stage('Remove Unused docker image'){
            steps{
                sh "docker rmi $registry:V$BUILD_NUMBER"
            }
        }

        stage('Deploy') {
            steps {
                withCredentials(bindings: [
                      string(credentialsId: 'kubernete-jenkis-server-account', variable: 'api_token')
                      ]){
                          sh 'kubectl --token $api_token --server https://192.168.49.2:8443 --insecure-skip-tls-verify=true apply -f deployment-billing-app-back-jenkins.yaml '
                      }
            }
        }


    }


}