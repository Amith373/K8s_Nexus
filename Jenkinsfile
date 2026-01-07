pipeline {
    agent any

    environment {
        PROJECT_KEY = "demo-project"
        VERSION     = "4.0.0.${BUILD_NUMBER}"
        IMAGE_NAME  = "demo-project"
    }

    tools {
        maven 'MAVEN'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Amith373/demo-use-repo.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }

        stage('Sonar Analysis') {
            steps {
                withSonarQubeEnv('Sonarqube') {
                    sh """
                        mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=${PROJECT_KEY} \
                        -Dsonar.projectName=${PROJECT_KEY}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
            post {
                success {
                    echo "Quality Gate Passed"
                }
                failure {
                    echo "Quality Gate Failed"
                }
                aborted {
                    echo "Quality Gate Aborted"
                }
            }
        }

        stage('Build with Version') {
            steps {
                sh """
                    mvn clean install \
                    -Drevision=${VERSION}
                """
            }
        }

        stage('Nexus Artifactory Upload') {
            steps {
                stage('Nexus Artifactory Upload') { steps { nexusArtifactUploader artifacts: [[artifactId: 'my-app', classifier: '', file: 'target/my-app-1.0-SNAPSHOT.jar', type: '.jar']], credentialsId: '66547aac-4104-464f-ac9f-2057b94190a5', groupId: 'com.mycompany.app', nexusUrl: '43.204.217.231:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-release', version: ${VERSION} }
            }
        }
    }
}
