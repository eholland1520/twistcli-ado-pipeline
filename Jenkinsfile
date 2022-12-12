 
 pipeline {
     environment {
      SECR = credentials("prisma_secret")
      TWISTLOCK_URL = credentials("prisma_url")
      ARTIFACTORY_SECR = credentials("artifactory_secret")
      ARTIFACTORY_URL = credentials("artifactory_url")
    }
   
    agent docker 
    stages {
        stage('Install TwistCli') { 
            steps {
                 sh '''#!/bin/bash
                  echo "hello world"
                  echo "Install TwistCLI"
                  ls
                  echo $SECR_USER
                  echo $SECR_CONSOLEURL
                  echo $SECR_PASSWORD
                  curl -k -O -u $SECR_USR:$SECR_PSW $TWISTLOCK_URL/api/v1/util/twistcli
                  pwd
                  ls
                  env
                  chmod a+x twistcli;
                '''
            }
        }
     stage('Pull Public Docker Image') { 
            steps {
                  sh '''#!/bin/bash
                  docker pull bitnami/rabbitmq
                '''
            }
        }
     stage('Image Vulnerability Scan') { 
            steps {
                  sh '''#!/bin/bash
                  echo "Start Image Scan"
                  ./twistcli images scan --details --address ${TWISTLOCK_URL} --u ${SECR_USR} -p ${SECR_PSW} bitnami/rabbitmq
                '''
            }
        }
        stage('Runtime - Image Analysis Sandbox') { 
            steps {
                  sh '''#!/bin/bash
                  echo "Start Image Scan"
                  sudo ./twistcli sandbox --analysis-duration 30s --address ${TWISTLOCK_URL} --u ${SECR_USR} -p ${SECR_PSW} bitnami/rabbitmq
                '''
            }
        }
     stage('Push Verified Image to Artifactory') { 
            steps {
                  sh '''#!/bin/bash
                  docker login ${ARTIFACTORY_URL} -u ${ARTIFACTORY_SECR_USR} -p ${ARTIFACTORY_SECR_PSW}
                  docker tag bitnami/rabbitmq ${ARTIFACTORY_URL}/prismaclouddev-docker-local/prisma-rabbitmq:latest
                  docker push ${ARTIFACTORY_URL}/prismaclouddev-docker-local/prisma-rabbitmq:latest
                '''
            }
        }
    }
 }
