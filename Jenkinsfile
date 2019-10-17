void buildAndRelease() {
  try {
    sh "jx step helm build"
    sh "jx step helm release"
  } catch(err) {
    echo hudson.Functions.printThrowable(err)
  }
}

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
           buildAndRelease()
          }
          dir('nuxeo-web-ui') {
            buildAndRelease()
          }
          dir('nuxeo-mongodb') {
            buildAndRelease()
          }
          dir('nuxeo-postgresql') {
            buildAndRelease()
          }
          dir('nuxeo-redis') {
            buildAndRelease()
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
