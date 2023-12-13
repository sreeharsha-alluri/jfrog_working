pipeline {
  agent any
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
  }
  environment {
    CI = true
    ARTIFACTORY_ACCESS_TOKEN = credentials('jfrog_token')
  }
  stages {
    stage('Build') {
      steps {
        script {
          sudo sh './mvnw clean install'
        }
      }
    }
    stage('Upload to Artifactory') {
      agent any
      steps {
        script {
          docker.image('releases-docker.jfrog.io/jfrog/jfrog-cli-v2:2.2.0').inside {
            sh 'jfrog rt upload --url http://44.212.5.222:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} target/demo-0.0.1-SNAPSHOT.jar java-web-app/'
          }
        }
      }
    }
  }
}
