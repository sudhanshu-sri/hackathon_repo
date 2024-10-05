pipeline {
   agent any 
    environment {     
    DOCKERHUB_CREDENTIALS= credentials('DOCKERHUB')     
  } 
   stages{
    
       stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
      stage('Checkout from Git') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/sibanando/hackathon_repo.git'
            }
        }
      stage('Trivy File Scan') {
            steps {
                
                    sh 'trivy fs . > trivyfs.txt'
                
            }
        }
    stage('SAST - Sonar') {
           environment {
               scannerHome = tool 'sonar-scanner';
           }
           steps {
             withSonarQubeEnv(credentialsId: 'sonar-token', installationName: 'sonar-server') {
               sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectName=hackathon-proj -Dsonar.projectKey=hackathon-proj "
             }
           }
       }
     stage('Quality Check') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token' 
                }
            }
        }
	// stage('OWASP Dependency-Check Scan') {
     //       steps {
     //          
     //               dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
     //               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      //         
      //      }
       // }
    
    // Building Docker images
   
     // Uploading Docker images into Docker Hub
      stage('Building image') {
        steps{    
           script {
             sh 'docker system prune -f'
             sh 'docker container prune -f'
             sh 'docker build -t  sibhanayak/pythonapp:$BUILD_NUMBER .'
            }
        }
      }
    
   stage('Login to Docker Hub') {         
      steps{                            
    	sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'                 
    	echo 'Login Completed'
      }
   }
    
    
    
    
    // Running Docker container, make sure port 8096 is opened in 
     stage('Docker Push') {
       steps{
         script {
           sh 'docker push sibhanayak/pythonapp:$BUILD_NUMBER'
         }
      }
    }
    stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image sibhanayak/pythonapp:$BUILD_NUMBER > trivyimage.txt' 
            }
        }
        stage('Checkout2 from Git') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/sibanando/hackathon_repo.git'
            }
        }
        stage('Update Deployment file') {
            environment {
                GIT_REPO_NAME = "hackathon_repo"
                GIT_USER_NAME = "sibanando"
            }
            steps {
		
                    withCredentials([string(credentialsId: 'github', variable: 'GITHUB_TOKEN')]) {
                        sh '''
                            git config user.email "sibhanayak@hotmail.com"
                            git config user.name "sibanando"
                            BUILD_NUMBER=${BUILD_NUMBER}
                            echo $BUILD_NUMBER
							sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" deployment.yaml
                            git add deployment.yaml
                            git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                            git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                        '''
                    }
            }
        }
     
}
post{
    always {  
      sh 'docker logout'           
    }      
  }  
}
  
