# sample-java-maven
DevOps demo

Jenkins Pipeline script(Groovy)

node {
  git url: 'https://github.com/shanthshivam/sample-java-maven.git'
  def mvnHome = tool 'M3'
  bat "${mvnHome}\\bin\\mvn -B verify"
    step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
}


Multi-stage Sonar analysis

// multistage
node {

    stage("Source"){
        git url: 'https://github.com/shanthshivam/sample-java-maven.git'       
    }
    stage("Build"){
        def mvnHome = tool 'M3'
        bat "${mvnHome}\\bin\\mvn -B verify"    
    }
    stage('SonarQube Analysis') {
        def mvnHome = tool 'M3'
        withSonarQubeEnv() {
            bat "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=java-maven"
        }
    }
    stage("Test"){
        step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
    }
    stage("Packaging"){
        step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
    }
           
}

// multistage
pipeline {
    agent any

        stages {
            stage('Source') {
                steps {
                    git url: 'https://github.com/shanthshivam/sample-java-maven.git'
                }
            }
            stage('Build') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        bat "${mvnHome}\\bin\\mvn -B verify"
                    }
                }
            }
            stage('SonarQube Analysis') {
                steps {
                    script {
                        def mvnHome = tool 'M3'
                        withSonarQubeEnv() {
                            bat "${mvnHome}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=java-maven"
                        }
                    }
                }
            }
            stage('Test') {
                steps {
                    step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
                }
            }
            stage('Packaging') {
                steps {
                    step([$class: 'ArtifactArchiver', artifacts: '**/target/*.jar', fingerprint: true])
                }
            }
            
            stage ("Artifactory Publish"){
                steps{
                    script{
                            def server = Artifactory.server 'artifactory'
                            def rtMaven = Artifactory.newMavenBuild()
                            //rtMaven.resolver server: server, releaseRepo: 'jenkins-devops', snapshotRepo: 'jenkins-devops-snapshot'
                            rtMaven.deployer server: server, releaseRepo: 'jenkins-devops', snapshotRepo: 'jenkins-devops-snapshot'
                            rtMaven.tool = 'M3'
                            
                            def buildInfo = rtMaven.run pom: '$workspace/pom.xml', goals: 'clean install'
                            rtMaven.deployer.deployArtifacts = true
                            rtMaven.deployer.deployArtifacts buildInfo

                            server.publishBuildInfo buildInfo
                    }
                }
        }
        }
}
