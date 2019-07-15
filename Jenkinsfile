node {
    def mvnHome
    def server=Artifactory.server 'Jfrog'
//Clone the repo from Git    
        stage('Preparation') { // for display purposes
	    git 'https://github.com/naveenbg1982/Java-Mysql-Simple-Login-Web-application.git'
	     mvnHome = tool 'Maven'
        }
//Passing through quality checks	
	stage('sonarqube'){
	   withSonarQubeEnv("sonar"){
		  sh'mvn clean package sonar:sonar'
	   }
	}
//Waiting for response frm SonarQube
	stage("Quality Gate") {
		timeout(time: 1, unit: 'HOURS') {
			def qg = waitForQualityGate()
			if (qg.status != 'OK') {
			error "Pipeline aborted due to quality gate failure: ${qg.status}"
			currentBuild.status='FAILURE'
			}
		}
	}
//Building the application with Maven	
	stage('Build') {
	// Run the maven build
		withEnv(["MVN_HOME=$mvnHome"]) {
			sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean install package'
		}
	}
//Upload to Artifactory repo to save the war file	
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
//Download from Artifactory into the VM		
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
//Building a dcoker image for the application   
	stage('docker image build') {
	   sh 'docker build -t naveenbg1982/simplejavalogin:${BUILD_NUMBER} .'
	}
//Pushing to the Docker hub
	stage('docker image push') {
	   sh 'docker push naveenbg1982/simplejavalogin:${BUILD_NUMBER}'
	}
//Deplying into the VM where docker-compse is running	
	stage('SCP'){
		sh "scp ../../pipeline1/LoginWebApp.war devopsmachine@mgmhy4755dns1.eastus2.cloudapp.azure.com:/opt/tomcat/webapps/"
	}
    
}
	
	
  
