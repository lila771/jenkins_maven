pipeline {
	environment {
		MAGE = readMavenPom().getArtifactId()
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
				script {
					def server = Artifactory.server('artifactory2')
					def rtMaven = Artifactory.newMavenBuild()
					def buildInfo = Artifactory.newBuildInfo()
					rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
					rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
				}
			}
		}
		stage('Install'){
			steps{
				script { 
					rtMaven.run pom: 'jenkins_project/pom.xml', goals: 'clean install'
				}
			}
		}
		stage('Deploy'){
			steps{
				script {
					rtMaven.deployer.deployArtifacts buildInfo
				}
			}
		}
		stage('Publish build info'){
			steps{
				script {
					server.publishBuildInfo buildInfo
				}
			}
		}
	}
}
