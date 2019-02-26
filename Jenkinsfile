pipeline {
  agent {
    label "jenkins-jx-base"
  }
  environment {
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
  }
  stages {
    stage('Build nuxeo helm chart in SNAPSHOT version') {
      when {
        branch 'master'
      }
      steps {
        container('jx-base') {
          dir('nuxeo') {
            sh "jx step helm build"
            sh "jx step helm release"
          }
          dir('nuxeo-mongodb') {
            sh "jx step helm build"
            sh "jx step helm release"
          }
          dir('nuxeo-postgresql') {
            sh "jx step helm build"
            sh "jx step helm release"
          }
          dir('nuxeo-redis') {
            sh "jx step helm build"
            sh "jx step helm release"
          }
        }
      }
    }
  }
  post {
    always {
      cleanWs()
    }
  }
}
