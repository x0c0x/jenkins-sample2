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
        stage('Slack Notification'){
             steps{
                script{
                //slackSend channel: '#jenkins-build', color: 'Good', message: 'Welcome to Jenkins', teamDomain: 'x0c0x', tokenCredentialId: 'slacknotification'
                slackSend color: '#BADA55', message: 'Hello, World!', channel: '#jenkins-build'                
                }
             }    
        }
    }
}
