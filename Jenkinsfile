def COLOR_MAP = [
  'SUCCESS': 'good',
  'FAILURE': 'danger',
  ]

pipeline {
    
	agent any
	tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin123'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.88.118'
        NEXUSPORT = '8081'
        NEXUS_REPOSITORY = 'vprofile-release'
	    NEXUS_GRP_REPO = 'vprofile-maven-group'
        NEXUS_LOGIN = 'nexuslogin'
        SONARSERVER = 'sonarserver'
        SONARSCANER = 'sonarscanner'
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn -s  settings.xml -DskipTests install'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }
        stage('UNIT TEST'){
            steps {
                sh 'mvn -s  settings.xml test'
            }
        }
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn -s  settings.xml checkstyle:checkstyle'
            }    
        }
        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool "${SONARSCANER}"
          }

          steps {
            withSonarQubeEnv("${SONARSERVER}") {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile-rework \
                   -Dsonar.projectName=vprofile-rework \
                   -Dsonar.host.url=http://172.31.90.154 \
                   -Dsonar.login=66038f7b82bc3d2319a57f0a3ff49564ad1613ab
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
             timeout(time: 10, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: true
             }
            }
         }
         stage('NexusArtifactUploaderJob') {
            steps {
             nexusArtifactUploader(
             nexusVersion: 'nexus3',
             protocol: 'http',
             nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
             groupId: 'QA',
             version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
             repository: "${RELEASE_REPO}",
             credentialsId: "${NEXUS_LOGIN}",
             artifacts: [
                [artifactId: 'vproapp',
                classifier: '',
                file: 'target/vprofile-v2.war',
                type: 'war']
             ]
          )
        }
    }
 }
  post {
            always {
                echo 'Slack Notifications.'
                slackSend channel: '#vprofilecicd',
                    color: COLOR_MAP[currentBuild.currentResult],
                    message: "*${currentBuild.currentResult}:*Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}