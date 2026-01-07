pipeline {
    agent any
    environment{
        PROJECT_KEY="demo-project"
        VERSION="1.0.${env.BUILD_NUMBER}"
        IMAGE_NAME="demo-project"
    }
    
    
    tools{
        maven 'MAVEN'
    }

    stages {
        stage('Git Checkout') {
            steps {
               git branch: 'main', url: 'https://github.com/Amith373/demo-use-repo.git'
            }
        }
         stage('Build') {
            steps {
               sh 'mvn clean install'
            }
        }
         stage('sonar analysis'){
            steps{
                withSonarQubeEnv('Sonarqube'){
                   sh """ mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=${PROJECT_KEY} \
                    -Dsonar.projectName=${PROJECT_KEY}
                    """
                }
            }
         }
         stage('Quality gate'){
            steps{
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
                  post{
                     notBuilt{
                           echo "Quality Gate success"
                        }
                     aborted{
                           echo "Quality Gate Failed"
                     }
                }
             stage('Build'){
            steps{
                sh """ mvn clean install \
                       -Drevision=${VERSION} """
            }
        }
        stage('Nexus-artifactory'){
            steps{
                nexusArtifactUploader artifacts: [[artifactId: 'my-app', classifier: '', file: 'target/com.mycompany.app-4.0.0.jar', type: '.jar']], credentialsId: '66547aac-4104-464f-ac9f-2057b94190a5', groupId: 'com.mycompany.app', nexusUrl: '43.204.217.231:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-release', version: "${VERSION}"}"
            }
        }
         }
    }    
}
