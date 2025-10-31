pipeline {
    agent any
    /*tools {
        jdk 'myJDK'
        maven 'myMVN'
    }
    environment {
        registryCredential = 'ecr:ap-south-1:awscreds'
        appRegistry = '590184026146.dkr.ecr.ap-south-1.amazonaws.com/vprofileappimg'
        vprofileRegistry = 'https://590184026146.dkr.ecr.ap-south-1.amazonaws.com'
        cluster = 'vprofile'
        service = 'vprofileappsvc'
    }*/

    stages {
        stage ('Fetch Code') {
            steps {
                git branch: 'docker',
                url: 'https://github.com/reyman84/vprofile-project.git'
            }
        }

        stage ('Build') {
            steps {
                sh 'mvn install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving it...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }
        /*stage ('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage ('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
        stage ('Sonar Code Analysis') {
            environment {
                scannerHome = tool 'sonar6.2'
            }
            steps {
                withSonarQubeEnv('sonarserver') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                       -Dsonar.projectName=vprofile \
                       -Dsonar.projectVersion=1.0 \
                       -Dsonar.sources=src/ \
                       -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                       -Dsonar.junit.reportsPath=target/surefire-reports/ \
                       -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                       -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
					// http://172.31.41.234:8080/sonarqube-webhook
                }
            }
        }
		stage('Upload Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.47.231:8081',
                    groupId: 'QA',
                    version: "${env.BUILD_ID}",
                    repository: 'vprofile-repo',
                    credentialsId: 'nexuslogin',
                    artifacts: [
                        [artifactId: 'vproapp',
                        classifier: '',
                        file: 'target/vprofile-v2.war',
                        type: 'war']
                    ]
                )
            }
        }*/
	}
}
