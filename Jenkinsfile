pipeline{
  agent any
  tools {
	  maven "Maven3"
	  jdk "OracleJDK11"
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
	  NEXUS_LOGIN = 'nexuslogin'
	  SONARSERVER = 'Sonarserver'
	  SONARSCANNER = 'Sonarscanner'
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
			  
  }
}
			  
	
