pipeline {
	agent {
	    docker { 
		    reuseNode true
		    image 'maven:3.5.2-jdk-8-alpine' 
		    def server
		    def rtMaven
		    def buildInfo
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
		stage('Artifactory configuration'){
			steps{
				withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
					script {
						server = Artifactory.server('artifactory2')
						rtMaven = Artifactory.newMavenBuild()
						buildInfo = Artifactory.newBuildInfo()
						buildInfo.env.capture = true
						rtMaven.tool = 'maven352'
						env.JAVA_HOME = '/usr/lib/jvm/java-8-oracle/jre'
						rtMaven.resolver server: server, releaseRepo: 'libs-release', snapshotRepo: 'libs-snapshot'
						rtMaven.deployer server: server, releaseRepo: 'libs-release-local', snapshotRepo: 'libs-snapshot-local'
					}
				}
			}
		}
		stage('Install') {
			steps{
				withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
					script {
						rtMaven.tool = 'maven352'
						env.JAVA_HOME = '/usr/lib/jvm/java-8-oracle/jre'
						rtMaven.run pom: 'pom.xml', goals: 'clean install -Dmaven.repo.local=.m2'
					}
				}
			}
		}
		stage('Deploy') {
			steps{
				withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
					script {
						rtMaven.tool = 'maven352'
						env.JAVA_HOME = '/usr/lib/jvm/java-8-oracle/jre'
						rtMaven.deployer.deployArtifacts buildInfo
					}
				}
			}
		}
		stage('publish BuildInfo') {
			steps{
				withEnv( ["PATH+MAVEN=${tool mvn_version}/bin"] ) {
					script {
						rtMaven.tool = 'maven352'
						env.JAVA_HOME = '/usr/lib/jvm/java-8-oracle/jre'
						server.publishBuildInfo buildInfo
					}
				}
			}
		}
	}
}
