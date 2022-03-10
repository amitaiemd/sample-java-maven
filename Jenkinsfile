pipeline {
  agent {
    node {
      label 'java-maven'
    }

  }
  stages {
    stage('build') {
      steps {
        sh '''def mvnHome = tool \'M3\'
bat "${mvnHome}\\\\bin\\\\mvn -B verify"'''
      }
    }

  }
}