#!groovy
def artifactId="ccdscore"
def groupId=com.dbs.ccdt"
//def gitrepourl = ""
def gitInfo = {}

pipeline {
  agent {
    label 'MAVEN'
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
    
    
			  
}
}
