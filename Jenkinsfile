pipeline {
  agent {
    kubernetes {
      defaultContainer 'jnlp'
      yamlFile 'pod-template.yml'
    }
  }
  environment {
    ORG = 'redpillanalytics'
    APP_NAME = 'gradle-confluent'
    CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    DOCKER_REGISTRY_ORG = 'rpa-jenkins-2'
  }
  stages {
    stage('Release') {
      when {
        branch 'master'
      }
      steps {
        container('gradle') {
          // ensure we're not on a detached head
          sh "git checkout master"
          sh "git config --global credential.helper store"
          sh "jx step git credentials"

          // so we can retrieve the version in later steps
          sh "echo \$(jx-release-version) > VERSION"
          sh "jx step tag --version \$(cat VERSION)"
        }
      }
    }
    stage('Build') {
      when {
        branch 'PR-*'
      }
      environment {
        PREVIEW_VERSION = "0.0.0-SNAPSHOT-$BRANCH_NAME-$BUILD_NUMBER"
        PREVIEW_NAMESPACE = "$APP_NAME-$BRANCH_NAME".toLowerCase()
        HELM_RELEASE = "$PREVIEW_NAMESPACE".toLowerCase()
      }
      steps {
        container('gradle') {
          sh "clean build"
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