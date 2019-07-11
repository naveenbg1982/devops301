node {
   def mvnHome
   def server=Artifactory.server 'Jfrog'
   stage('Preparation') { // for display purposes
      // Get some code from a GitHub repository
      git 'https://github.com/naveenbg1982/Java-Mysql-Simple-Login-Web-application.git'
      // Get the Maven tool.
      // ** NOTE: This 'M3' Maven tool must be configured
      // **       in the global configuration.           
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
if (isUnix()) {
sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore clean package'
} else {
bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
}
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
  
