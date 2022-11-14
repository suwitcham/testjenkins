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
                /* sh '/var/lib/jenkins/tioscript/upimage.sh' */
            }
        }
        stage('Validate') {
            steps {
                sh '/var/lib/jenkins/tioscript/CS-status.sh'   
            }
        }
    }
}
