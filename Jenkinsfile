pipeline {
    agent {
	    docker { 
		    reuseNode true
		    image 'maven:3.5.2-jdk-8-alpine' 
		}
	}
	environment {
	    IMAGE = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }
	
    stages {
	    stage('Build') {
            steps {
			    checkout scm
                sh 'mvn clean findbugs:findbugs package'               
            }
		}	
	    stage('Artifactory'){
		    steps {
			    def server = Artifactory.server('artifactory2')
				def rtMaven = Artifactory.newMavenBuild()
				def buildInfo = rtMaven.run pom: 'jenkins_project/pom.xml', goals: 'clean install'
				rtMaven.tool = 'maven352'
				rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
				rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
				rtMaven.deployer.deployArtifacts buildInfo
				server.publishBuildInfo buildInfo
			}					
				
		}
    }		
	post {
        success {
            archiveArtifacts(artifacts: '**/target/*.jar', allowEmptyArchive: true)
        }
    }
}
