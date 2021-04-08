pipeline {

      agent none

      stages {
          stage('Build Branch Master or Demo') {
              when {
                    anyOf {
                          branch "master"
                          branch "demo"
                    }
              }
              agent {
                  node {
                      label 'salf-aws'
                      customWorkspace '/home/ubuntu/jenkins/multi-branch/'
                  }
              }

              steps {
                  script {
                        if (env.BRANCH_NAME=="master" || env.BRANCH_NAME=="demo"){
                          sh "cp serviceConfig_${env.BRANCH_NAME}.json serviceConfig.json"
                          sh "aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 362085406009.dkr.ecr.us-west-2.amazonaws.com"
                          sh "docker build -t blueeye_admin ."
                        }
                  }
              }
          }

          stage('Build Branch Development') {
              when {
                    branch 'development'
              }

              agent {
                  node {
                      label 'SALF_Local'
                      customWorkspace '/home/ubuntu/jenkins/multi-branch/'
                  }
              }

              steps {
                  script {
                        if (env.BRANCH_NAME=="development"){
                          sh "cp serviceConfig_${env.BRANCH_NAME}.json serviceConfig.json"
                          sh "docker build -t blueeye_admin-${env.BRANCH_NAME} ."
                        }
                  }
              }
          }

          stage('Deploy Branch Master or Demo') {
              when {
                    anyOf {
                          branch "master"
                          branch "demo"
                    }
              }
              agent {
                  node {
                      label 'salf-aws'
                      customWorkspace '/home/ubuntu/jenkins/multi-branch/'
                  }
              }

              steps {
                  script {
                        if (env.BRANCH_NAME=="master"|| env.BRANCH_NAME=="demo"){
                              CONTAINER_ID = sh(returnStdout: true, script: "docker ps -a | grep blueeye_admin | awk '{print \$1}'").trim()
                              sh "echo ${CONTAINER_ID}"
                              if (CONTAINER_ID != ''){
                                  sh "docker container rm ${CONTAINER_ID} --force"
                              }
                              sh "docker tag blueeye_admin:latest 362085406009.dkr.ecr.us-west-2.amazonaws.com/blueeye_admin:${env.BRANCH_NAME}"
                              sh "aws ecr batch-delete-image --repository-name blueeye_admin --image-ids imageTag=${env.BRANCH_NAME}"
                              sh "docker push 362085406009.dkr.ecr.us-west-2.amazonaws.com/blueeye_admin:${env.BRANCH_NAME}"
                              sh "docker rmi -f blueeye_admin:latest"
                              sh "docker rmi -f 362085406009.dkr.ecr.us-west-2.amazonaws.com/blueeye_admin:${env.BRANCH_NAME}"
                              sh 'return'
                        }
                  }
              }
          }

          stage('Deploy Branch Development') {
              when {
                    branch 'development'
              }
              agent {
                  node {
                      label 'SALF_Local'
                      customWorkspace '/home/ubuntu/jenkins/multi-branch/'
                  }
              }

              steps {
                  script {
                        if (env.BRANCH_NAME=="development"){
                              CONTAINER_ID = sh(returnStdout: true, script: "docker ps -a | grep blueeye_admin-${env.BRANCH_NAME} | awk '{print \$1}'").trim()
                              sh "echo ${CONTAINER_ID}"
                              if (CONTAINER_ID != ''){
                                  sh "docker container rm ${CONTAINER_ID} --force"
                              }
                              sh "docker run -d -e BRANCH_SOURCE_DEPLOY=${env.BRANCH_NAME} --name blueeye_admin-${env.BRANCH_NAME} --network=host blueeye_admin-${env.BRANCH_NAME}"
                              sh 'return'
                        }
                  }
              }
          }
      }

}
