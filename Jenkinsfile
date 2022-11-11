/* Requires the Docker Pipeline plugin */
pipeline {
    agent { docker { image 'maven:3.8.6-openjdk-11-slim' } }
    stages {
        stage('build') {
            steps {
                sh 'mvn --version'
            }
        }
        stage('Test') {
            steps {
                sh 'docker save maven:3.8.6-openjdk-11-slim | docker run -e TENABLE_ACCESS_KEY=d5e755cf6742182f305b3313d458a512e2bed63a707091afbeb20bbda93e30e2 -e TENABLE_SECRET_KEY=eac92d40a73a1fee5198f6e64c426cd9c0726312af2b9a9b886f8341cf8c9f1f -e IMPORT_REPO_NAME=JenkinsPipe -i tenableio-docker-consec-local.jfrog.io/cs-scanner:latest inspect-image maven'
            }
        }
        stage('Validate') {
            steps {
                sh '/var/lib/jenkins/tioscript/CS-status.sh'   
            }
        }
    }
}
