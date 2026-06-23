node {

   
    def mavenHome=tool name: "maven-3.9.9"

    try {
notifyBuild('STARTED')
  stage('Git checkout')
  {
    git branch: 'development', credentialsId: 'aa528cef-cbd8-4ef2-8c34-cb5cff2be464', url: 'https://github.com/kkdevopsb4/maven-web-app-project-kk-funda.git'
  }
    stage('COMPILE')
  {
   sh "${mavenHome}/bin/mvn clean compile"
  }
  stage('Build')
  {
   sh "${mavenHome}/bin/mvn clean package"
  }
   /*stage('SQ REPORT')
  {
   sh "${mavenHome}/bin/mvn clean sonar:sonar"
  } */
     stage('Deploy to nexus')
  {

   sh "${mavenHome}/bin/mvn clean deploy"
  }
   stage('Deploy to Tomcat') {
        echo "Deploying WAR file using curl..."

        sh """
            curl -u kkfunda:password \
            --upload-file /var/lib/jenkins/workspace/jio-Scripted-dev-PL/target/maven-web-application.war \
            "http://43.204.114.41:8080/manager/text/deploy?path=/maven-web-application&update=true"
        """
    }

	} catch (e) {
    // If there was an exception thrown, the build failed
    currentBuild.result = "FAILED"
    throw e
  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)
  }
}  // node close

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'BLUE'
    colorCode = '#281c90'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#00FF00'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#airtel-dev')
}
