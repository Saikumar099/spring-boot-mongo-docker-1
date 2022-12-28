pipeline {
    agent none
      tools{
       git 'Git'
       maven 'maven3.8.6'
       //sonarqubescanner 'SonarQubeScanner'
      }
      environment {     
              //DOCKERHUB_CREDENTIALS= credentials('docker-hub') 
              AWS_ACCOUNT_ID="948406862378"
              AWS_DEFAULT_REGION="us-west-1"
              IMAGE_REPO_NAME="spring-boot-app"
              IMAGE_TAG="latest"
              REPOSITORY_URI ="${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
        } 
       stages{
           stage('checkout code') {
               agent {
                    label 'master'
               }
               steps{
               checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Saikumar099/spring-boot-mongo-docker-1.git']]])  
               stash 'source'
               }
           }
            stage('maven build') {
               agent {
                    label 'master'
               }
               steps{
                     sh 'mvn package'
                }
           }
           stage('sonarqube report') {
               agent {
                    label 'node-1'
               }
               environment{
                   scannerHome = tool 'SonarQubeScanner'
               }
               steps{
                    unstash 'source'
                    withSonarQubeEnv('Sonarqube') { 
                        // sh "${"SonarQubeScanner"}/bin/sonar-scanner" 
                         // sh '''$scannerHome/bin/sonar-scanner 
                        //-Dsonar.host.url=http://54.193.191.66:9000 
                        //-Dproject.settings=sonar-project.properties
                        //-Dsonar.projectKey=spring-app 
                        //-Dsonar.projectName=spring-app 
                        //-Dsonar.java.binaries=target/classes'''
                       sh 'mvn clean install sonar:sonar -Dsonar.host.url=http://13.57.214.44:9000 -Dproject.settings=sonar-project.properties -Dsonar.projectKey=spring-app -Dsonar.projectName=spring-app || true'
                    }
                }
            }
            stage('upload artifacts to nexus') {
               agent {
                    label 'node-1'
              }
              steps{
                   nexusArtifactUploader artifacts: [[artifactId: 'spring-boot-mongo', 
                   classifier: '', 
                   file: 'target/spring-boot-mongo-1.0.war', 
                   type: 'war']], 
                   credentialsId: 'nexus', 
                   groupId: 'com.mt', 
                   nexusUrl: '13.57.214.44:8081', 
                   nexusVersion: 'nexus3', 
                   protocol: 'http', 
                   repository: 'spring-app', 
                   version: '1.0'
               } 
           }
           stage('creating tomcat image with webapp') {
              agent {
                    label 'node-1'
              }
               steps{
                    //unstash 'source'
                    sh 'docker build -t saikumar099/spring-app:$BUILD_NUMBER .'
              }
           }
	       stage('Logging into AWS ECR') {
              steps {
                 script {
                sh """aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"""
                }
             }
          }
         stage('pushing image to ECR') {
             //agent {
               //     label 'Docker Server'
             //}
	        steps{
		      script{
		       //sh 'aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/y0r0a3j7'
		      // sh “aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com”
		       //sh 'docker build -t ecr-demo .'
		       sh """docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"""
		      // sh 'docker tag ecr-demo:latest public.ecr.aws/y0r0a3j7/ecr-demo:latest'
		      // sh 'docker push public.ecr.aws/y0r0a3j7/ecr-demo:latest'
			 sh """docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"""
			     }
		     }
	     }  	  
       /* stage('Login to Docker Hub') { 
           agent {
              label 'Docker Server'
              }
              steps{                       	
              	sh 'echo $DOCKERHUB_CREDENTIALS_PSW | sudo docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'                		
	            echo 'Login Completed'      
                   }           
                }   
           stage('pushing image to dockerhub registry') {
              agent {
                    label 'Docker Server'
              }
                steps{   
                  // withDockerRegistry(credentialsId: 'docker-hub', url: 'https://hub.docker.com/repository/docker/saikumar099/java-web-app') {
                     sh 'docker push saikumar099/java-web-app:$BUILD_NUMBER'
                     echo 'Push Image Completed'
                     //}  
                  }
             }*/
             stage('deploying image to k8s') {
                agent {
                    label 'node-1'
                }
                steps{
                    sh '''
                    aws eks --region us-west-1 update-kubeconfig --name eks-cluster
                    kubectl apply -f springBootMongo.yml
                    '''
                }
            }
       }
}
