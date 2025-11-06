/*
Plugins:        Git Integration, Maven Integration, Nexus Artifact Uploader, SonarQube Scanner, Build Timestamp, Slack Notification
Tools:          JDK 17, Maven 3.9, SonarQube Scanner 4.7.0.2747
Credentials:    sonartoken, slacktoken, gitlogin, nexuslogin
Other Setup:    Git Webhook, SonarQube Webhook, SonarQube Quality Gate, Slack configuration, Git Repository

Slack Configuration: 
Email:      devopspractice@myyahoo.com
Workspace:  Accenture (accenture-3hn2465)
Channel:    devops_practices
Token:      sLxuMSHJ3uCrisWYGPZPFyow
*/

/*def COLOR_MAP = [
    'SUCCESS': 'good',          // 'good' means green in slack
    'FAILURE': 'danger'         // 'danger' means red in slack
]*/

pipeline {
    agent any

    tools {
        maven "myMVN"
        jdk "myJDK_ubuntu"
    }

    environment {
        // Nexus Repositories
        RELEASE_REPO = 'vprofile-release'       // Maven 2 (hosted) repository
        SNAP_REPO = 'vprofile-snapshot'         // Maven 2 (hosted) repository
        CENTRAL_REPO = 'vpro-maven-central'     // Maven 2 (hosted) proxy           https://repo1.maven.org/maven2/
        NEXUS_GRP_REPO = 'vpro-maven-group'     // Maven 2 (hosted) group

        // Nexus configuration
        NEXUSIP = '172.21.2.25'                 // Always change when new servers are launched
        NEXUSPORT = '8081'
        NEXUS_LOGIN = 'nexuslogin'

        // SonarQube configuration
        SONARSCANNER = 'sonarscanner'
        SONARSERVER = 'sonarserver'
    }

    stages {

        stage ('AuditTools') {
            steps {
                sh '''
                mvn --version
                java -version
                jenkins --version
                git --version
                '''
            }

        }
        stage('Build') {
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
                script {
                    env.APP_VERSION = "${env.BUILD_ID}-" + sh(
                        script: "date -u +%Y%m%d%H%M%S",
                        returnStdout: true
                    ).trim()

                    echo "Generated APP_VERSION = ${env.APP_VERSION}"
                }
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'QA',
                    version: "${APP_VERSION}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        type: 'war',
                        file: 'target/vprofile-v2.war']
                    ]
                )
            }
        }

        stage('Deploy to Staging') {
            steps {
                echo "Deploying version ${APP_VERSION} to staging..."
                sh """
                    ansible-playbook \
                    -i ansible/inventory/stage \
                    ansible/deploy.yml \
                    --extra-vars "version=${APP_VERSION}"
                """
            }
        }

        stage('Manual Approval') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    input message: "Approve deployment to PRODUCTION?",
                           ok: "Proceed",
                           submitter: "Ramandeep Singh,admin"
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "Deploying version ${APP_VERSION} to production..."
                sh """
                    ansible-playbook \
                    -i ansible/inventory/prod \
                    ansible/deploy.yml \
                    --extra-vars "version=${APP_VERSION}"
                """
            }
        }
    }
    
    /*post {
        always {
            echo "Slack Notification"
            slackSend channel: '#devops_practices', 
            color: COLOR_MAP[currentBuild.currentResult],
            message: "*${currentBuild.currentResult}:* - Job ${env.JOB_NAME} Build ${env.BUILD_NUMBER} \n  More info at: ${env.BUILD_URL}"
        }
    }*/
}
