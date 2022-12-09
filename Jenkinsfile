 
 pipeline {
     environment {
      /*
       * Uses a Jenkins credential called "FOOCredentials" and creates environment variables:
       * "$FOO" will contain string "USR:PSW"
       * "$FOO_USR" will contain string for Username
       * "$FOO_PSW" will contain string for Password
       */
      SECR = credentials("prismacloudsecrets")
    }
    agent any 
    stages {
        stage('Install TwistCli') { 
            steps {
                  sh '''#!/bin/bash
                  echo "hello world"
                  echo "Install TwistCLI"
                  ls
                  curl -k -O -u $SECR_USER:$SECR_PASSWORD $SECR_CONSOLEURL/api/v1/util/twistcli
                  pwd
                  ls
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
                  ./twistcli images scan --details --address ${CONSOLEURL} --u ${USER} -p ${PASSWORD} bitnami/rabbitmq
                '''
            }
        }
        stage('Runtime - Image Analysis Sandbox') { 
            steps {
                  sh '''#!/bin/bash
                  echo "Start Image Scan"
                  sudo ./twistcli sandbox --analysis-duration 30s --address ${CONSOLEURL} --u ${USER} -p ${PASSWORD} bitnami/rabbitmq
                '''
            }
        }
        stage('Push Verified Image to Artifactory') { 
            steps {
                  sh '''#!/bin/bash
                  docker login ${ARTIFACTORYURL} -u ${ARTIFACTORYUSER} -p ${ARTIFACTORYTOKEN}
                  docker tag bitnami/rabbitmq ${ARTIFACTORYURL}/prismaclouddev-docker-local/prisma-rabbitmq:latest
                  docker push ${ARTIFACTORYURL}/prismaclouddev-docker-local/prisma-rabbitmq:latest
                '''
            }
        }
    }
 }
