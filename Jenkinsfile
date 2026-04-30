pipeline {
    agent {
        label 'agent-1'
    } 
    environment {
        appVersion = ""
        REGION = "us-east-1"
        ACC_ID = "838180513114"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    stages {
        stage('Read package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "package.json version ${appVersion}"
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    sh """
                        npm install
                    """
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    withAWS(credentials: 'aws-cerds', region: 'us-east-1') {
                       sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                       """
                    }
                }
            }
        }

    }

    post { 
        always { 
            echo 'I will always say Hello again!'
        }

        success { 
            echo 'If success say ..I will always say Hello again!'
        }

        
        failure { 
            echo 'If failure say ..I will always say stage is failure!'
        }
    }

}