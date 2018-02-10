pipeline {
	agent any
	environment {
		def mvn_version = 'maven352'
		ІMAGE = readMavenPom().getArtifactId()
		VERSION = readMavenPom().getVersion()
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
		stage('Artifactory configuration'){
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
