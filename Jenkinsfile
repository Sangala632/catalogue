pipeline {
    agent {
        label 'agent-1'
    } 
    options {
        timeout(time: 30, unit: 'MINUTES') 
        disableConcurrentBuilds()
    }
    stages{
        stage('Build') {
            steps {
                echo "Building the application"
            }
        }

        stage('Test') {
            steps {
                echo "Testing the build application"
            }
        }

        stage('Deploy') {
            input {
                message "Should we continue?"
                ok "Yes, we should."
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                }
            }
            steps {
                scrpit {
                    echo "Hello, ${PERSON}, nice to meet you"
                    echo "Deploying the application in vm "
                }
                
            }
        }

        stage('Deploy-2') {
            steps {
                echo "optional only"
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