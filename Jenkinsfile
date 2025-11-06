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
        jdk "myJDK"
    }

    environment {
        // Nexus Repositories
        RELEASE_REPO = 'vprofile-release'       // Maven 2 (hosted) repository
        SNAP_REPO = 'vprofile-snapshot'         // Maven 2 (hosted) repository
        CENTRAL_REPO = 'vpro-maven-central'     // Maven 2 (hosted) proxy           https://repo1.maven.org/maven2/
        NEXUS_GRP_REPO = 'vpro-maven-group'     // Maven 2 (hosted) group

        // Nexus configuration
        NEXUSIP = '172.21.2.161'                 // Always change when new servers are launched
        NEXUSPORT = '8081'
        NEXUS_LOGIN = 'nexuslogin'

        // SonarQube configuration
        SONARSCANNER = 'sonarscanner'
        SONARSERVER = 'sonarserver'
    }

    stages {

        stage ('Ansible Installation') {
            steps {
                sh '''
                    echo "Checking Ansible installation..."
                    if ! command -v ansible &> /dev/null
                    then
                        echo "Ansible not found, installing..."
                        sudo apt update
                        sudo apt install software-properties-common -y
                        sudo add-apt-repository --yes --update ppa:ansible/ansible
                        sudo apt install ansible -y
                    else
                        echo "Ansible is already installed."
                    fi
                    ansible --version
                '''
            }
        }

        stage ('AuditTools') {
            steps {
                auditTools()
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
				sonarPush()
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
		
		stage('Blue-Green Deploy - Staging') {
            steps {
                script {
                    sh """
                        ansible-playbook -i ansible/inventory/stage ansible/deploy.yml --extra-vars "version=${APP_VERSION} env=stage"
                    """
                }
            }
        }
		
		stage('Smoke Tests') {
            steps {
                sh 'curl -f http://staging.myapp.local/health'
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
		
		stage('Blue-Green Deploy - Production') {
            steps {
                script {
                    sh """
                        ansible-playbook -i ansible/inventory/prod ansible/deploy.yml --extra-vars "version=${APP_VERSION} env=prod"
                    """
                }
            }
        }
        
        stage('Post-Deploy Validation') {
            steps {
                sh 'curl -f http://prod.myapp.local/health'
            }
        }

        stage('Cleanup Old Artifacts') {
            steps {
                sh '''
                    echo "Cleaning up old artifacts in Nexus..."
                    # Add cleanup commands here
                '''
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

void auditTools () {
    sh '''
        mvn --version; 
        java -version
        jenkins --version
        git --version
        ansible --version
    '''
}

void sonarPush () {
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