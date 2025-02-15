pipeline { 
    agent any
    
    tools {
        maven 'maven3'
        jdk 'jdk17'
    }
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    stages {
        
        stage('Cleaning Workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('CodeCheckOutforUpdate') {
            steps {
           git branch: 'main', url: 'https://github.com/devopsenggr/Bank-App.git'
            }
        }
        
        stage('Compile') {
            steps {
            sh  "mvn compile"
            }
        }
        
        
        stage('Package') {
            steps {
                sh "mvn package"
            }
        }
                stage('Sonarqube Analysis') {
            steps {
                
                    withSonarQubeEnv('sonar-server') {
                        sh ''' $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Bank \
                        -Dsonar.projectKey=Bank \
                        -Dsonar.java.binaries=target'''
                    }
               
            }
        }
        
        
        stage('Trivy File Scan') {
            steps {
                    sh 'trivy fs . > trivyfs.txt'
                  }
        } 
        
      //  stage('publishArtifactoryToNexus') {
        //    steps {
          //      withMaven(globalMavenSettingsConfig: 'maven-settings', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {
            //            sh "mvn deploy"
              //  }
                
            //}
        //}
        
        stage('Docker BuildTag and push') {
            steps {
                script
                {
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker') 
                    {
                        sh'docker build -t awsd43/goldencatbankapp:${BUILD_NUMBER} .'
                        sh 'docker push awsd43/goldencatbankapp:${BUILD_NUMBER}'
                    }
                }
            }
        } 
        stage("TRIVY Image Scan") {
            steps {
                sh 'trivy image awsd43/goldencatbankapp:${BUILD_NUMBER} > trivyimage.txt' 
            }
        }
        stage('CodeCheckOutForUpdate') {
            steps {
           git branch: 'main', url: 'https://github.com/devopsenggr/Bank-App.git'
            }
        }
        stage('Update Deployment file') 
        {
            environment 
            {
                GIT_REPO_NAME = "Bank-App"
                GIT_USER_NAME = "devopsenggr"
            }
            steps 
            {
               
                withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                    sh '''
                    set +e
                    git config user.email "awstraining42@gmail.com"
                    git config user.name "devopsenggr"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    echo $BUILD_NUMBER
                    sed -i "s/goldencatbankapp:.*/goldencatbankapp: \${BUILD_NUMBER}/" deployment-service.yml
                    git add .
                    git commit -m "Update deployment Image to version \${BUILD_NUMBER}"
                    git push https://${github-token}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                    '''
                }
                
            }
        }
        
    }
}
