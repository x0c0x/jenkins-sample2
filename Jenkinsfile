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
	// Send slack msg with start build	    
		 slackSend channel: '#jenkins-build',
                  color: 'good',
                  message: "The pipeline ${currentBuild.fullDisplayName} is building..."
            }
        }
    }
     
	
	stages('SonarQube Analysis') {
		steps{
                script{

        def mvnHome =  tool name: 'apache-maven-3.6.1', type: 'maven'
        withSonarQubeEnv('sonar7') { 
          sh "${mvnHome}/bin/mvn sonar:sonar"
        	}
   	}	
		}
	}
		
	stages('Upload War To Nexus'){
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

      
        stages('Slack Notification'){
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
}
