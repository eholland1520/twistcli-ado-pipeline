pipeline {
    agent any 
    stages {
        stage('Install TwistCLI') { 
            steps {
                echo 'Install TwistCLI' 
                curl -k -O -u [USER:PASSWORD] [PRISMACONSOLEURL];
                chmod a+x twistcli;
            }
        }
        stage('Test') { 
            steps {
                echo 'Prisma Image Vulnerability Scan'
                sudo ./twistcli images scan --details --address [PRISMACONSOLEURL] --u [USER] -p [PASSWORD] prismaclouddev.jfrog.io/prismaclouddev-docker-local/rabbitmq:latest
            }
        }
        stage('Deploy') { 
            steps {
                echo 'Prisma Image Analysis Sandbox'  
                sudo ./twistcli sandbox --analysis-duration 15s --address [PRISMACONSOLEURL] --u [USER] -p [PASSWORD] prismaclouddev.jfrog.io/prismaclouddev-docker-local/rabbitmq:latest
            }
        }
    }
}
