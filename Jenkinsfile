node {
	def server
	def rtMaven
	def buildInfo
	
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
			server = Artifactory.server('artifactory2')
			rtMaven = Artifactory.newMavenBuild()
			buildInfo = Artifactory.newBuildInfo()
			rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
			rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
		}
		stage('Install'){
			rtMaven.run pom: 'jenkins_project/pom.xml', goals: 'clean install'
		}
		stage('Deploy'){
		rtMaven.deployer.deployArtifacts buildInfo
		}
		stage('Publish build info'){
			server.publishBuildInfo buildInfo
		}
	}	
}
