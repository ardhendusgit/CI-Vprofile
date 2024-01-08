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

	stages {

	    stage('Fetch code') {
            steps {
               git branch: 'ci-jenkins', url: 'https://github.com/ardhendusgit/CI-Vprofile.git'
            }

	    }

	    stage('Build'){
	        steps{
	           sh 'mvn install -DskipTests'
	        }

	        post {
	           success {
	              echo 'Now Archiving it...'
	              archiveArtifacts artifacts: '**/*.war'
	           }
	        }
	    }

	    stage('UNIT TEST') {
            steps{
                sh 'mvn test'
            }
        }

        stage('Checkstyle Analysis'){
        	steps{
        		sh 'mvn checkstyle:checkstyle'
        	}
        }

        stage('Sonar Analysis'){
        	environment{
        		scannerHome = tool 'sonar4.7'
        	}
        	steps{
                withSonarQubeEnv('sonar') {
                sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Vprofile \
                	-Dsonar.projectName=ardhendu-vprofile \
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
              }
            }
        }

        stage("UploadArtifact") {
        	steps{
        		    nexusArtifactUploader(
        				nexusVersion: 'nexus3',
        				protocol: 'http',
        				nexusUrl: '172.31.58.163:8081',
        				groupId: 'QA',
        				version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
        				repository: 'vprofile-repo',
        				credentialsId: 'nexuslogin',
        				artifacts: [
            			[	artifactId: 'ardhendu-vpro-app',
             				classifier: '',
             				file: 'target/vprofile-v2.war',
             				type: 'war'
             			]
        			]
     			)		
        	}
        }
	}

	post{
		  always {
          echo 'Slack Notifications.'
          slackSend channel: '#notifications',
          color: COLOR_MAP[currentBuild.currentResult],
          message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
      	}
	}
}