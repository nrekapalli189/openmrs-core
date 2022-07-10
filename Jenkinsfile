#!groovy
def artifactId="ccdscore"
def groupId=com.dbs.cct"
//def gitrepourl = ""
def gitInfo = {}

pipeline {
  agent {
    label 'DSLAVE2'
  }
  tools {
    maven 'maven-3.5.0'
    jdk 'JDK8'
  }
  options{
    skipDefaultCheckout()
  }
  
  stages {
    stage('Checkout from GIT') {
      steps {
        script {
          gitInfo = checkout scm
          print gitInfo
          //gitrepourl = "https://bitbucket.sgp.dbs.com:8443/dcifgit/scm/${env.APPCODE/${ENV.REPONAME}.git"
          nexus_version = env.BUILD_NUMBER+'.'+ sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          println nexus_version
          String br = gitInfo.GIT_BRANCH
          br = br.replaceAll("origin/","")
          gitInfo.GIT_BBRANCH = br
          echo gitInfo.GIT_BRANCH
          //echo gitInfo.GIT_URL
          String giturl = gitInfo.GIT_URL.toString()
          giturl = giturl.substring(0,giturl.lastIndexOf("REPO")-2)
          giturl = giturl + params.REPONAME + ".git"
          gitInfo.GIT_URL = giturl
          echo "$gitInfo.GIT_URL"
        }
      }
    }
    
    stage('Build with maven') {
		steps {
			script {
				sh '''
				pwd
				mvn -f pom.xml -U clean install -Dmaven.test.skip=true
				ls -ltr
			mkdir -p ccdscore
			mkdir -p ccdscore/ccdt_DB_SQL
			mkdir -p ccdscore/properties
			test -f ./target/ccds.war && co ./target/ccds.war ccdscore
			ls -ld ${WORKSPACE}/ccdt_DB_SQL
			cp -R ${WORKSPACE}/ccdt_DB_SQL ccdscore/
			ls -ld ${WORKSPACE}/properties
			ls -ld ${WORKSPACE}/properties ccdscore/
			zip -r ccdscore.zip ccdscore/
			pwd
			ls -lrt
			cd ${WORKSPACE}/deploymentProperties
			zip -r ${WORKSPACE}/deploymentProperties.zip *
			ls -lrt
			'''
			}
		}
	}
	  
	  stage('Nexus IQ Scan') {
		  steps {
			  script {
				  Map mp = [ commitID: gitInfo.GIT_COMMIT,
							branch: gitInfo.GIT_BRANCH,
							repourl: gitInfo.GIT_URL,
							iqProjectName: "${env.APPCODE}_${REPONAME}",
							mailto: "rameshbabu@dbs.com, nagabhushanamr@dbs.com",
							organization: "MAS_or_External",
							appCategory: "Hosted"
							]
				  performIQScan(mp)
			  }
		  }
	  }
	  stage('UnitTest') {
		  steps {
			  sh "mvn install -f pom.xml"
			  sh "cd tools; java PostCSVToConfluence ../ccdscore/target/jacoco-ut/jacoco.csv"
			  sh "ant -lib ./unittest -f build.xml uploadCoverageReport"
		  }
	  }
	  stage ('Jacoco') {
		  steps {
			  sh "ls -l ./target"
		  }
	  }
	  stage ("Sonar Scan") {
		  steps{
			  script {
				  Map mp =[commitID: gitInfo.GIT_COMMIT,
						   branch: gitInfo.GIT_BRANCH,
						   repourl: gitInfo.GIT_URL,
						   "sonar.projectKey": "CCDT_ccdsCore",
						   "sonar.projectName": "CCDT_ccdScore",
						   "sonar.sources": "src",
						   "sonar.exclusions": "**/*.jpg",
						   "sonar.java.binaries": ".",
						   "sonar.branch.name": gitInfo.GIT_BRANCH,
						   "sonar.junit.reportPaths": "target/surefire-reports",
						   qualityGateCheck: false
						   ]
				  peformSonarScan(mp)
			  }
		  }
	  }
	  
	  stage('Fortify Scan') {
		  steps {
			  script {
				  Map mp = [ commitID: gitInfo.GIT_COMMIT,
							branch: gitInfo.GIT_BRANCH,
							repourl: gitInfo.GIT_URL,
							fortifyProjectName: "${env.APPCODE}",
							fortifyVersionName: "{env.APPCODE}_ccdscore",
							buildCommand: "mvn -f pom.xml -U clean install -Dmaven.test.skip=true"
							]
				  performFortifyScan(mp)
			  }
		  }
	  }
	  stage('Upload Package to Nexus repo') {
		  steps {
			  nexusArtifactUploader artifacts: [[artifactId: artifactId, classifier: '', file: "{WORKSPACE}/ccdscore.zip", type: 'zip']],
				  credentialsId: 'nexusArtifactUploader',
				  groupId: groupId,
				  nexusUrl: 'nexusscimgmt.sgp.dbs.com:8443/nexus',
				  nexusVersion: 'nexus3',
				  protocol: 'https',
				  repository: 'CCDT',
				  version: nexus_version
			  nexusArtifactUploader artifacts: [[artifctId: "deploymentProperties", file: "${WORKSPACE}/deploymentProperties.zip", type: 'zip']],
				  credentialsId: 'nexusArtifactUploader',
				  groupId: groupId,
				  nexusUrl: 'nexusscimgmt.sgp.dbs.com:8443/nexus',
				  nexusVersion: 'nexus3',
				  protocol: 'https',
				  repository: 'CCDT',
				  version: nexus_version
		  }
	  }
  }
	post {
		always {
			script
			{
				def status = currentBuild.currentResult == 'SUCCESS' ? 'SUCCESS' : 'Fail'
				Map jiramap = [
					"projectkey" : "CCDTRA",
					"ENVIRONMENT" : "Build",
					"status" : status,
					"JIRA_KEY" : "${env.JIRA_KEY}",
					'AAPP_VERSION' : nexus_version,
					'CommitID' : gitInfo.GIT_COMMIT,
					'Nexus Artifct ID':artifactId,
					'Nexus Group ID':groupId,
					'Repo URL': gitInfo.GIT_URL,
					'addComment':"Build Status: " +status
					]
				updateJiraTicket(jiramap)
				deleteDir() /* clean up our workspace */
			}
		}
	}
			  
