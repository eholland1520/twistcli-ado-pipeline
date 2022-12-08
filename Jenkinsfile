 
 pipeline {
    agent any 
    stages {
      stage('Get Credentials') { 
        environment {
          SECRET_FILE_ID = credentials('prismacloudsecrets')
         }
            steps {
                  sh '''#!/bin/bash
                  echo "####DISPLAYING SECRET_FILE_ID####"
                  echo "Global property file: ${SECRET_FILE_ID}"

                  echo "#####Copying Global property file to src\\main\\resources#####"
                  bat "powershell Copy-Item ${SECRET_FILE_ID} -Destination src\\main\\resources"

                  echo "#####Deleting Local Global property file present in src\\main\\resources#####"
                  bat "powershell Remove-Item src\\main\\resources\\global-dev.properties"

                  echo "#####Renaming Global property file in Jenkins to global-dev.properties#####"
                  bat "powershell Rename-Item src\\main\\resources\\secret_file_jenkins global-dev.properties"
                '''
            }
        }
        stage('Install TwistCli') { 
            environment {
                USER = credentials('user')
                PASSWORD = credentials('password')
                CONSOLEURL = credentials('consoleurl')
            }
            steps {
                  sh '''#!/bin/bash
                  echo "hello world"
                  echo "Install TwistCLI"
                  ls
                  curl -k -O -u ${USER}:${PASSWORD} ${CONSOLEURL}/api/v1/util/twistcli
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
            environment {
                USER = credentials('user')
                PASSWORD = credentials('password')
                CONSOLEURL = credentials('consoleurl')
            }
            steps {
                  sh '''#!/bin/bash
                  echo "Start Image Scan"
                  ./twistcli images scan --details --address ${CONSOLEURL} --u ${USER} -p ${PASSWORD} bitnami/rabbitmq
                '''
            }
        }
        stage('Runtime - Image Analysis Sandbox') { 
            environment {
                USER = credentials('user')
                PASSWORD = credentials('password')
                CONSOLEURL = credentials('consoleurl')
            }
            steps {
                  sh '''#!/bin/bash
                  echo "Start Image Scan"
                  sudo ./twistcli sandbox --analysis-duration 30s --address ${CONSOLEURL} --u ${USER} -p ${PASSWORD} bitnami/rabbitmq
                '''
            }
        }
        stage('Push Verified Image to Artifactory') { 
            environment {
                ARTIFACTORYTOKEN = credentials('artifactorytoken')
                ARTIFACTORYUSER = credentials('artifactoryuser')
                ARTIFACTORYURL = credentials('artifactoryurl')
            }
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
