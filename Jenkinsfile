node {
    def mvnHome
    def server=Artifactory.server 'Jfrog'
    
        stage('Preparation') { // for display purposes
	    git 'https://github.com/naveenbg1982/Java-Mysql-Simple-Login-Web-application.git'
	     mvnHome = tool 'Maven'
        }
	stage('sonarqube'){
	   withSonarQubeEnv("sonar"){
		  sh'mvn clean package sonar:sonar'
	   }
	}

	stage("Quality Gate") {
		timeout(time: 1, unit: 'HOURS') {
			def qg = waitForQualityGate()
			if (qg.status != 'OK') {
			error "Pipeline aborted due to quality gate failure: ${qg.status}"
			currentBuild.status='FAILURE'
			}
		}
	}
	
	stage('Build') {
	// Run the maven build
		withEnv(["MVN_HOME=$mvnHome"]) {
			sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
		}
	}
   
	stage('docker image build') {
	   sh 'docker build -t naveenbg1982/simplejavalogin:${BUILD_NUMBER} .'
	}

	stage('docker image push') {
	   sh 'docker push naveenbg1982/simplejavalogin:${BUILD_NUMBER}'
	}

	stage('artifactory upload') {
		def uploadSpec = """{
		"files": [
		{
			"pattern": "target/LoginWebApp.war",
			"target": "repo1/"
		}
		]
		}"""
		server.upload(uploadSpec)
	}
		
	stage('artifactory download') {
		def downloadSpec = """{
		"files": [
		{
			"pattern": "repo1/LoginWebApp.war",
			"target": "/var/lib/jenkins/pipeline1/"
		}
		]
		}"""
		server.download(downloadSpec)
	}
	stage('SCP'){
		sh "scp ../../pipeline1/LoginWebApp.war devopsmachine@mgmhy4755dns1.eastus2.cloudapp.azure.com:/opt/tomcat/webapps/"
	}
    
}
	
	
  
