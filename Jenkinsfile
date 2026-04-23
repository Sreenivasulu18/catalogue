@Library('jenkins-shared-library') _

def configMap = [
    project: "roboshop",
    component: "catalogue"
]
 
// If branch is not equal to main, then run CI Pipeline
if ( ! env.BRANCH_NAME.equalsIgnoreCase('main') ){
    nodeJSEKSPipeline(configMap)
}
else {
    echo "Please follow the CR Process"
}
