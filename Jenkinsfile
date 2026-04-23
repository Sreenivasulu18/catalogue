@Library('jenkins-shared-library') _

def configMap = [
    project: "roboshop",
    component: "catalogue",
]

// if branch is not equal to main, then run CI pipeline 8
if ( ! env.BRANCH_NAME.equalsIgnoreCase('main') ){
    nodeJSEKSPipeline(configMap)
}
else {
    echo "Please follow the CR process"
}