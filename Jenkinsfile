pipeline {
  //parameters {
  //  string(name: 'WORK_FOLDER', defaultValue: 'container', description: 'Working folder')
  //  string(name: 'IMAGE_NAME', defaultValue: 'my_wordpress', description: 'Image name')
  //  string(name: 'IMPORT_REPO_NAME', defaultValue: 'gcr.io/wordpress-in-gks', description: 'Registry Name')
  //  booleanParam(name: 'PUSH_IMAGE', defaultValue: false , description: 'Push Image')
  //  booleanParam(name: 'PARAM_CHECK_POLICY', defaultValue: false , description: 'Blocking action')

  // }
  agent {
    kubernetes {
      // yamlFile 'container.yaml'
      yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu:focal
    command:
    - sleep
    args:
    - infinity
  - name: kaniko
    image: gcr.io/kaniko-project/executor:debug
    command:
    - sleep
    args:
    - infinity
  - name: cs-scanner
    image: tenableio-docker-consec-local.jfrog.io/cs-scanner
    imagePullPolicy: Always
    command:
    - sleep
    args:
    - infinity
  imagePullSecrets:
    - name: tenable-regcred
'''

      defaultContainer 'kaniko'
    }
  }
  stages {
    stage('Build Image') {
      steps {
        container('kaniko') {
          dir("${env.WORKSPACE}/${params.WORK_FOLDER}") {
            sh '''
              /kaniko/executor \
                    --context `pwd` \
                    --tarPath ./${IMAGE_NAME}.tar \
                    --no-push \
                    --destination ${IMAGE_NAME}
            '''
            sh 'echo "Image Built" ; ls -la ./${IMAGE_NAME}.tar'
          }
        }
      }
    }
    stage('Scan Image') {
      steps {
        container('cs-scanner') {
          dir("${env.WORKSPACE}/${params.WORK_FOLDER}") {
            withCredentials([
              string(credentialsId: 'TenableAccessKey', variable: 'TENABLE_ACCESS_KEY'),
              string(credentialsId: 'TenableSecretKey', variable: 'TENABLE_SECRET_KEY')])
            {
              sh '''
                test $PARAM_CHECK_POLICY = true && export CHECK_POLICY=true
                export IMAGE_NAME_FULL=${IMAGE_NAME}:${GIT_COMMIT}
                cat ./${IMAGE_NAME}.tar | /opt/tenable/consec/bin/cs-scanner inspect-image ${IMAGE_NAME_FULL}
              '''
            }
          }
        }
      }
    }
    stage('Push Image') {
      when {
        expression {
          return params.PUSH_IMAGE
        }
      }
      steps {
        container('kaniko') {
          dir("${env.WORKSPACE}/${params.WORK_FOLDER}") {
            withCredentials([
              file(credentialsId: 'gcp-wordpress-in-gks-creds-tf', variable: 'GOOGLE_APPLICATION_CREDENTIALS')])
            {
              sh 'cat ${GOOGLE_APPLICATION_CREDENTIALS} > /kaniko/config.json'
              sh '''
                /kaniko/executor \
                      --context `pwd` \
                      --destination ${IMPORT_REPO_NAME}/${IMAGE_NAME}:${GIT_COMMIT}
              '''
              sh '''
                GOOGLE_APPLICATION_CREDENTIALS=/kaniko/config.json /kaniko/executor \
                      --context `pwd` \
                      --destination ${IMPORT_REPO_NAME}/${IMAGE_NAME}:latest
              '''
            }
          }
        }
      }
    }
  }
  post {
    failure {
      container('ubuntu') {
        dir("${env.WORKSPACE}/${params.WORK_FOLDER}") {
          withCredentials([
            string(credentialsId: 'TenableAccessKey', variable: 'TENABLE_ACCESS_KEY'),
            string(credentialsId: 'TenableSecretKey', variable: 'TENABLE_SECRET_KEY')])
          {
            sh '''
              apt-get update
              apt-get install -y jq curl
            '''
            sh '''
              curl -v -H "X-ApiKeys: accessKey=$TENABLE_ACCESS_KEY; secretKey=$TENABLE_SECRET_KEY" \
                https://cloud.tenable.com/container-security/api/v2/reports/${IMPORT_REPO_NAME}/${IMAGE_NAME}/${GIT_COMMIT} \
                -o scan-report.json
            '''

            sh '''
              cat scan-report.json | jq
            '''
          }
        }
      }
      archiveArtifacts artifacts: 'scan-report.json', fingerprint: true
    }
  }
}
