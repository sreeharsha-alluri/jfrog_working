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
                script {
                    // Use a temporary variable for ARTIFACTORY_ACCESS_TOKEN
                    def accessToken = ARTIFACTORY_ACCESS_TOKEN
                    // Use Docker image directly
                    def dockerImage = 'releases-docker.jfrog.io/jfrog/jfrog-cli-v2:2.2.0'
                    
                    // Check if sudo is required
                    def useSudo = sh(script: "docker run --rm -v /var/lib/jenkins/workspace/new:/workspace ${dockerImage} /bin/sh -c 'test -w /workspace'", returnStatus: true) != 0

                    // Run the Docker command with or without sudo
                    def dockerCommand = useSudo ? 'sudo docker' : 'docker'
                    
                    // Execute commands inside the Docker container
                    sh """
                        ${dockerCommand} pull ${dockerImage}
                        ${dockerCommand} run --rm -v \$PWD:/workspace ${dockerImage} /bin/sh -c "cd /workspace && echo Printing current directory: && ls -al && echo Uploading artifact to JFrog... && jfrog rt upload --url http://44.201.254.41:8082/artifactory/ --access-token ${accessToken} target/demo-0.0.1-SNAPSHOT.jar example-repo-local/"
                    """
                }
            }
        }
    }
}
