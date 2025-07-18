// Plugins: Git Integration, Maven Integration, Nexus Artifact Uploader, SonarQube Scanner, Build Timestamp, Slack Notification

def COLOR_MAP = [
    'SUCCESS': 'good',          // 'good' means green in slack
    'FAILURE': 'danger'         // 'danger' means red in slack
]

pipeline {
    agent any

    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }

    environment {
        // Nexus Repositories
        RELEASE_REPO = 'vprofile-release'       // Maven 2 (hosted) repository
        SNAP_REPO = 'vprofile-snapshot'         // Maven 2 (hosted) repository
        CENTRAL_REPO = 'vpro-maven-central'     // Maven 2 (hosted) proxy
        NEXUS_GRP_REPO = 'vpro-maven-group'     // Maven 2 (hosted) group

        // Nexus configuration
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'Khalsa_1699'
        NEXUSIP = '172.21.2.134'
        NEXUSPORT = '8081'
        NEXUS_LOGIN = 'nexuslogin'

        // SonarQube configuration
        SONARSCANNER = 'sonarscanner'
        SONARSERVER = 'sonarserver'
    }

    stages {
        stage('Build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving the build artifacts"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Unit Tests') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload artifact to Nexus') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${env.Build_ID}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        type: 'war',
                        file: 'target/vprofile-.war']
                    ]
                )
            }
        }
    }
    post {
        always {
            echo "Slack Notification"
            slackSend channel: '#devops_practices', 
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* - Job ${env.JOB_NAME} Build ${env.BUILD_NUMBER} \n  More info at: ${env.BUILD_URL}"
        }
    }
}