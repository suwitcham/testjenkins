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
                sh './upimage.sh'
                sh 'echo stage test'
            }
        }
        stage('Validate') {
            steps {
                sh './CS-status.sh'   
            }
        }
    }
}
