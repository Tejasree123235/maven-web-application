node{
    echo "the build id is:${BUILD_ID}"
    echo "the build number is : ${BUILD_NUMBER}"
echo " the workspace is :${WORKSPACE}"
echo "the jenkins home is :${JENKINS_HOME}"
echo " the jenkins url is :${JENKINS_URL}"
echo " the job name is :${JOB_NAME}"
def mavenHome = tool name: 'maven3.9.9'
properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '5', daysToKeepStr: '', numToKeepStr: '5')), pipelineTriggers([pollSCM('* * * * *')])])

try{
stage('CheckOutCode'){
sendslacknotification('STARTED')
git branch: 'development', credentialsId: 'd6becbe3-5aa7-4a66-9915-ffe714eac8e5', url: 'https://github.com/Tejasree123235/maven-web-application.git'
}

stage('Build'){
sh "${mavenHome}/bin/mvn clean package"
}

stage('ExecuteSonarQubeReport'){
sh "${mavenHome}/bin/mvn clean sonar:sonar"
}

stage('UploadArtifactsIntoNexus'){
sh "${mavenHome}/bin/mvn clean deploy"
}
stage('uploadthefileintomcat'){
    sshagent(['c1846823-c69d-42af-8882-875e2bde77da']) {
       sh "scp -o StrictHostKeyChecking=no target/maven-web-application.war ec2-user@172.31.0.88:/opt/apache-tomcat-9.0.102/webapps"
}
}

}
catch(e){
    currentBuild.result="FAILURE"
throw e
}
finally{
    sendslacknotification(currentBuild.result)
}
}
def sendslacknotification(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESSFUL'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESSFUL') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary)
}
