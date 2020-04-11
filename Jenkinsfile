pipeline {
    agent any
    tools {
        maven 'mavenbuild'
    }
    options {
        buildDiscarder logRotator(daysToKeepStr: '5', numToKeepStr: '7')
    }
    stages{
        stage('Build'){
            steps{
                 sh script: 'mvn clean package'
                 archiveArtifacts artifacts: 'target/*.war', onlyIfSuccessful: true
            }
        }
        stage('Upload War To Nexus'){
            steps{
                script{

                    def mavenPom = readMavenPom file: 'pom.xml'
                    def nexusRepoName = mavenPom.version.endsWith("SNAPSHOT") ? "simpleapp-snapshot" : "simpleapp-release"
                    nexusArtifactUploader artifacts: [
                        [
                            artifactId: 'simple-app', 
                            classifier: '', 
                            file: "target/simple-app-${mavenPom.version}.war", 
                            type: 'war'
                        ]
                    ], 
                    credentialsId: 'nexuspw', 
                    groupId: 'in.javahome', 
                    nexusUrl: '192.168.1.100:8081', 
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: nexusRepoName, 
                    version: "${mavenPom.version}"
                    }
            }
        }


 /**       
        stage('Slack Notification'){
             steps{
                script{
                //slackSend channel: '#jenkins-build', color: 'Good', message: 'Welcome to Jenkins', teamDomain: 'x0c0x', tokenCredentialId: 'slacknotification'
               // slackSend color: 'good', message: 'Build is successfully completed', channel: '#jenkins-build'
              //  slackSend (color: 'good', channel: '#jenkins-build', message: "Completed: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
    
    def currentBuildStatus = 'Back to normal'
    def message = """
        *Jenkins Build*
        Job name: `${env.JOB_NAME}`
        Build number: `#${env.BUILD_NUMBER}`
        Build status: `${currentBuild.result}`
        Build details: <${env.BUILD_URL}/console|See in web console>
    """.stripIndent()

    slackSend(color: 'good', message: message)
}          
                    
                    
                    

                }
             } 
*/
//
/**
* Notification in slack to give the status of the build
* @param currentBuildStatus : status of current build
* @param previousBuildStatus : status of the previous build
*/
def notifyBuild(String currentBuildStatus, String previousBuildStatus)
{
	echo "NotifyBuild [previousBuildStatus:${previousBuildStatus},currentBuildStatus:${currentBuildStatus}]."
  
  	// build status of null means successful
  	currentBuildStatus = currentBuildStatus ?: 'SUCCESS'
  	previousBuildStatus = previousBuildStatus ?: 'SUCCESS'

  	// we set back to normal if we are in success and last wasn't
	if(previousBuildStatus != 'SUCCESS' && currentBuildStatus == 'SUCCESS')
	{
		currentBuildStatus = 'Back to normal'
	}

	//notification text
	def jobName = java.net.URLDecoder.decode("${env.JOB_NAME}", "UTF-8");
	def subject = "${jobName} - #${env.BUILD_NUMBER} ${currentBuildStatus}"
  	def summary = "${subject} (<${env.BUILD_URL}|Open>)"

  	//colors
  	if (currentBuildStatus == 'STARTED' || currentBuildStatus == 'UNSTABLE') 
	{
		//Yellow
		colorCode = '#FFFF00'
	} 
  	else if (currentBuildStatus == 'SUCCESS' || currentBuildStatus == 'Back to normal') 
	{
		//Green
		colorCode = '#00FF00'
	}
	else 
	{
		//Red
		colorCode = '#FF0000'
	}

	// we notify only errors and back to normal
	if(currentBuildStatus != 'STARTED' && currentBuildStatus != 'SUCCESS')
	{
		slackSend (color: colorCode, message: summary)
	}
//
        }
    }
}
