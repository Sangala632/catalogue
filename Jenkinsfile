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
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
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

        stage('Unit Testing') {
            steps {
                script {
                    sh """
                        echo "unit tests"
                    """
                }
            }
        }

        stage('Sonar scan') {
            environment {
                scannerHome = tool 'sonarqube-8.0'
            }
            steps {
                script {
                    withSonarQubeEnv(installationName:'sonarqube-8.0') { // 'My SonarQube Server' is the installationName
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
        // enable webhook in sonarqube server and wait for result
        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true }
            }
        }

        stage('Check Dependabot Alerts') {
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GITHUB_TOKEN')]) {
                    script {
                        def response = sh(
                            script: """
                                curl -sf \
                                -H "Accept: application/vnd.github+json" \
                                -H "Authorization: Bearer ${GITHUB_TOKEN}" \
                                -H "X-GitHub-Api-Version: 2022-11-28" \
                                "https://api.github.com/repos/daws-84s/catalogue/dependabot/alerts?state=open&per_page=100"
                            """,
                            returnStdout: true
                        ).trim()

                        def alerts = readJSON text: response

                        def criticalOrHigh = alerts.findAll { alert ->
                            def severity = alert?.security_advisory?.severity?.toLowerCase()
                            return severity == "critical" || severity == "high"
                        }

                        if (criticalOrHigh.size() > 0) {
                            error "❌ Found ${criticalOrHigh.size()} critical/high vulnerabilities. Fix and re-run."
                        }

                        echo "✅ No critical or high vulnerabilities found. Pipeline passed."
                    }
                }
            }
        }
        stage('Docker Build') {
            steps {
                script {
                    withAWS(credentials: 'aws-cerds', region: "${REGION}") {
                        sh """
                            aws ecr get-login-password --region ${REGION} | \
                            docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com

                            docker build -t ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker push ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }

        stage('Trigger Deploy') {
            when {
            expression { params.deploy }
            }
            steps {
                build job: 'catalogue-cd',
                parameters: [
                    string(name: 'appVersion', value: "${appVersion}"),
                    string(name: 'deploy_to', value: 'dev')
                ],
                wait: false, // VPC will not wait for SG pipeline completion
                propagate: false // even SG fails VPC will not be effected
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