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
          // Ensure mvnw script has executable permissions
          sh 'sudo chmod +x mvnw'
          // Regenerate Maven Wrapper files
          sh 'mvn -N io.takari:maven:wrapper'
          // Run the Maven build
          sh './mvnw clean install'
        }
      }
    }
    stage('Upload to Artifactory') {
      steps {
        script {
          // Use a Docker agent for running the JFrog CLI
          docker.image('releases-docker.jfrog.io/jfrog/jfrog-cli-v2:2.2.0').inside {
            // Run JFrog CLI command inside Docker container
            sh "jfrog rt upload --url http://44.212.5.222:8082/artifactory/ --access-token ${ARTIFACTORY_ACCESS_TOKEN} target/demo-0.0.1-SNAPSHOT.jar java-web-app/"
          }
        }
      }
    }
  }
}
