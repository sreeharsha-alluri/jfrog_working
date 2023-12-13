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
                    sh 'chmod +x mvnw'
                    // Regenerate Maven Wrapper files
                    sh 'mvn -N io.takari:maven:wrapper'
                    // Run the Maven build
                    sh './mvnw clean install'
                }
            }
        }
        stage('Upload to Artifactory') {
            steps {
                // Use a Docker agent for running the JFrog CLI
                script {
                    // Use a temporary variable for ARTIFACTORY_ACCESS_TOKEN
                    def accessToken = ARTIFACTORY_ACCESS_TOKEN
                    container('releases-docker.jfrog.io/jfrog/jfrog-cli-v2:2.2.0') {
                        // Run JFrog CLI command inside Docker container
                        sh """
                            echo "Printing current directory:"
                            pwd
                            ls -al
                            echo "Uploading artifact to JFrog..."
                            jfrog rt upload --url http://44.212.5.222:8082/artifactory/ --access-token \${accessToken} target/demo-0.0.1-SNAPSHOT.jar test/
                        """
                    }
                }
            }
        }
    }
}
