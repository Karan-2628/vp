pipeline{
  agent any
  tools {
	  maven "Maven3"
	  jdk "OracleJDK8"
  }
  environment {
	  SNAP_REPO = 'vprofile-snapshot'
	  NEXUS_USER = 'admin'
	  NEXUS_PASS = 'K@ran2628'
	  RELEASE_REPO = 'vprofile-release'
	  CENTRAL_REPO = 'vpro-maven-central'
	  NEXUSIP = '172.31.95.117'
	  NEXUSPORT = '8081'
	  NEXUS_GRP_REPO = 'vpro-maven-group'
	  NEXUS_LOGIN = '4ddc38e4-f5f0-4d68-81e4-d02b4064c310'
	  SONARSERVER = 'Sonarserver'
	  SONARSCANNER = 'Sonarscanner'
	  registryCredential = 'ecr:us-east-1:awscred'
	  appRegistry  = '610894416511.dkr.ecr.us-east-1.amazonaws.com/vpfileappimg'
	  vprofileRegistry = 'https://610894416511.dkr.ecr.us-east-1.amazonaws.com'
  }
  stages {
	  stage('Build'){
		  steps {
			  sh 'mvn -s settings.xml -DskipTests install'
		  }
		  post {
			  success {
				  echo "Archieving Artifacts"
				  archiveArtifacts artifacts: '**/*.war'
			  }
		  }
	  }
	  stage('Test'){
		  steps {
			  sh 'mvn -s settings.xml test'
		  }
	  }
	  stage('Checkstyle Analysis'){
		  steps {
			  sh 'mvn -s settings.xml checkstyle:checkstyle'
		  }
	  }
	  stage('Sonar Analysis'){
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
	  stage('Upload Arti'){
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
	  stage("Build image") {
		  steps {
			  script {
				  dockerImage = docker.build( appRegistry +":$BUILD_NUMBER", "./Docker-files/Dockerfile" )
			  }
		  }
	  }
	  stage("Upload to ECR") {
		  steps{
			  script {
				  docker.withRegistry( vprofileRegistry, registryCredential ) {
					  dockerImage.push("$BUILD_NUMBER")
					  dockerImage.push('latest')
				  }
			  }
		  }
	  }
  }
}
