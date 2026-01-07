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
                nexusArtifactUploader artifacts: [[artifactId: 'calculator-java', classifier: '', file: "target/calculator-java-${VERSION}.jar", type: 'jar']],
                    credentialsId: 'nexus-cred', groupId: 'com.example', nexusUrl: '16.16.104.141:30003', nexusVersion: 'nexus3', protocol: 
                    'http', repository: 'maven-releases', version: "${VERSION}"
            }
        }
         }
    }    
}
