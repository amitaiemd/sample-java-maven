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
