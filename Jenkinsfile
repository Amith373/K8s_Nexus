pipeline {
    agent any

    tools {
        maven 'maven-3'   // Must match Jenkins Global Tool Configuration
    }

    environment {
        PROJECT_KEY           = "java-k8s"
        NEXUS_URL             = "http://13.232.91.33:32000"
        NEXUS_REPO_SNAPSHOT   = "maven-snapshots"
        NEXUS_REPO_RELEASE    = "maven-test"
        PROJECT_VERSION       = ""
    }

    stages {

        stage('Checkout SCM') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Amith373/K8s_Nexus.git'
            }
        }

        stage('Set Project Version') {
            steps {
                script {
                    PROJECT_VERSION = sh(
                        script: "mvn help:evaluate -Dexpression=project.version -q -DforceStdout",
                        returnStdout: true
                    ).trim()

                    echo "📦 Project Version: ${PROJECT_VERSION}"
                }
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean verify'
            }
        }

        stage('JaCoCo Coverage') {
            steps {
                sh 'mvn jacoco:report'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-K8s') {
                    sh """
                        mvn sonar:sonar \
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
        }

        stage('Deploy to Nexus') {
            steps {
                script {
                    def isSnapshot = PROJECT_VERSION.endsWith("-SNAPSHOT")
                    def repoUrl = isSnapshot ?
                        "${NEXUS_URL}/repository/${NEXUS_REPO_SNAPSHOT}/" :
                        "${NEXUS_URL}/repository/${NEXUS_REPO_RELEASE}/"

                    echo "🚀 Deploying ${PROJECT_VERSION} to ${repoUrl}"

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'nexus-creds',
                            usernameVariable: 'NEXUS_USER',
                            passwordVariable: 'NEXUS_PASS'
                        )
                    ]) {

                        writeFile file: 'temp-settings.xml', text: """
<settings>
  <servers>
    <server>
      <id>nexus</id>
      <username>${env.NEXUS_USER}</username>
      <password>${env.NEXUS_PASS}</password>
    </server>
  </servers>
</settings>
"""

                        sh """
                            mvn deploy -s temp-settings.xml \
                            -DaltDeploymentRepository=nexus::default::${repoUrl}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline succeeded: Quality Gate passed & artifact deployed."
        }

        failure {
            echo "❌ Pipeline failed. Please check the Jenkins logs."
        }

        always {
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            junit '**/target/surefire-reports/*.xml'
        }

        cleanup {
            deleteDir()
        }
    }
}
