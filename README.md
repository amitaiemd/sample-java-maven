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
