pipeline {
   
    agent any
    
    tools {
        maven 'maven'
        jfrog 'jfrog-cli'
    } 
    
    environment {
		DOCKER_IMAGE_NAME = "hshanka.jfrog.io/frog-docker/spring-petclinic-hs:$BUILD_NUMBER"
	}
    
    stages {
       
       stage('Cloning Git') {
         steps {
             checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '1915d014-8308-45ab-bec7-bb5c7bb2a0e1', url: 'https://github.com/hshanka/spring-petclinic.git']])
            }
        }
        
        stage('Static Code Analysis') {
            steps {
                withSonarQubeEnv('') {
                    sh "mvn clean verify sonar:sonar -Dsonar.projectKey=spring-petclinic -Dsonar.projectName='spring-petclinic'"
                }
            }
        }

        stage('Compile Code, Run Tests, Build App') {
                steps {
                    sh 'mvn clean package'
                }
        } 

        stage('Build Docker Image') {
                steps {
                    script {
                        docker.build("$DOCKER_IMAGE_NAME", '-f Dockerfile .')
                    }
                }
        } 
            
        stage('Scan and push image') {
                steps {
                
                        // Scan Docker image for vulnerabilities using JFrog Xray
                        jf 'docker scan $DOCKER_IMAGE_NAME'

                        // Push final docker image to Artifactory
                        jf 'docker push $DOCKER_IMAGE_NAME'
                    
                }
        }

    }
}
