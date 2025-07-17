pipeline {
    agent any
    tools {
        maven "MAVEN3.9"
        jdk "JDK17"
    }
    
    environment {
        RELEASE_REPO = 'vprofile-release'
        SNAP_REPO = 'vprofile-snapshot'
        CENTRAL_REPO = 'vpro-maven-central'
		NEXUS_GRP_REPO = 'vpro-maven-group'
		NEXUS_USER = 'admin'
		NEXUS_PASS = 'Khalsa_1699'
		NEXUSIP = '172.21.2.128'
		NEXUSPORT = '8081'
        NEXUS_LOGIN = 'nexuslogin'
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
                sh 'mvn -s settings.xml checkstyle:check'
            }
        }
    }
}