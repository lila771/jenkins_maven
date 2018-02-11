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
		def mvn_version = 'maven352'
	}
	stages {
		stage('Build') {
			steps {
				checkout scm
				sh 'mvn clean findbugs:findbugs package'
			}
			post {
				success {
				archiveArtifacts(artifacts: '**/target/*.jar', allowEmptyArchive: true)
				}
			}
		}	
		stage('Artifactory deploy'){
			steps{
				withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
					script {
						def server = Artifactory.server('artifactory2')
						def rtMaven = Artifactory.newMavenBuild()
						def buildInfo = Artifactory.newBuildInfo()
						rtMaven.tool = 'maven352'
						rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
						rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
						rtMaven.run pom: 'pom.xml', goals: 'clean install'
						rtMaven.deployer.deployArtifacts buildInfo
						server.publishBuildInfo buildInfo
					}
				}
			}
		}
	}
}
