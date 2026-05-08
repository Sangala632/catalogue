@Library('jenkins-shared-library') _

def configMap = [
    PROJECT : "roboshop",
    COMPONENT : "catalogue"
]

if(! env.BRANCH_NAME.equalsIgnoreCase('main')){ //if not equls to main
    nodejsEKSPipeline(configMap) //bydefault it will , call function inside this pipeline
}
else{
    echo "Please process with PROD process"
}
