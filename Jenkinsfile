pipeline {
    // These are pre-build sections S 
    agent {
        node {
            label 'AGENT-1'
        }
    }
    environment {
        COURSE = "jenkins"
        appVersion = ""
        ACC_ID = "167819473967"
        PROJECT = "roboshop"
        COMPONENT = "catalogue"
    }
    options {
        timeout(time: 10, unit: 'MINUTES')
        disableConcurrentBuilds()
    }
  
    // this is build section 
    stages {
        stage('Read Version') {
            steps {
                script{
                    def packageJSON = readJSON file: 'package.json'
                    appVersion = packageJSON.version 
                    echo "app version: ${appVersion}"
                }
            }
        }
        stage('Install Dependencies') {   
            steps {
                script{
                    sh """
                        npm install
                    """
                }
            }
        }
        stage('Unit Test') {
            steps {
                script {
                    sh """
                        npm test 
                    """
                }
            }
        }
        // Here you need to select scanner tool and send the analysis to server
        // stage('Sonar Scan') {
        //     environment {
        //         def scannerHome = tool 'sonar-8.0'
        //     }
        //     steps {
        //         script{
        //             withSonarQubeEnv('sonar-server'){
        //                 sh "${scannerHome}/bin/sonar-scanner"
        //             }
        //         }
        //     }
        // }
        // stage('Quality Gate'){
        //     steps{
        //         timeout(time: 1, unit: 'HOURS') {
        //             // wait for the quality gate status
        //             // abortPipeline: true will fail the jenkins job if the quality gate is 'FAILED'
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }
        stage('Dependabot Security Gate') {
            environment {
                GITHUB_OWNER = 'Sreenivasulu18'
                GITHUB_REPO  = 'catalogue'
                GITHUB_API   = 'https://api.github.com'
                GITHUB_TOKEN = credentials('GITHUB_TOKEN')
            }
            steps {
                script{
                    
                    sh """
                    echo "Fetching Dependabot alerts..."

                    response=$(curl -s \
                        -H "Authorization: token $GITHUB_TOKEN" \
                        -H "Accept: application/vnd.github+json" \
                        "$GITHUB_API/repos/$GITHUB_OWNER/$GITHUB_REPO/dependabot/alerts?per_page=100")

                    echo "$response" > dependabot_alerts.json

                    high_critical_open_count=$(echo "${response}" | jq '[.[] 
                        | select(
                            .state == "open"
                            and (.security_advisory.severity == "high"
                                or .security_advisory.severity == "critical")
                        )
                    ] | length')
                            
                    echo "Open HIGH/CRITICAL alerts count: $high_critical_count"

                    if [ "${high_critical_open_count}" -gt 0 ]; then
                        echo "❌ Blocking pipeline due to OPEN HIGH/CRITICAL Dependabot alerts"
                        echo "Affected dependencies:"
                        echo "$response" | jq '.[] 
                        | select(.state=="open" 
                        and (.security_advisory.severity=="high" 
                        or .security_advisory.severity=="critical"))
                        | {dependency: .dependency.package.name, severity: .security_advisory.severity, advisory: .security_advisory.summary}'
                        exit 1
                    else
                        echo "✅ No OPEN HIGH/CRITICAL Dependabot alerts found"
                    fi
                    """
                }
            }
        }

        stage('Build Image') {
            steps {
                script{
                    withAWS(region:'us-east-1',credentials:'aws-creds') {
                        sh """
                            aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com
                            docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                            docker images 
                            docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }
        stage('Deploy') {
            // input {
            //     message "Should we continue?"
            //     ok "Yes, we should."
            //     submitter "alice,bob"
            //     parameters {
            //         string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
            //     }
            // }
            when {
                expression { "$params.DEPLOY" == "true" }
            }
            steps {
                script{
                    sh """
                        echo "Deploying"
                    """
                }
            }
        }
    }
    
    post {
        always {
            echo 'I will always say Hello again!' 
            cleanWs()
        }
        success {
            echo 'I will run if success'
        }
        failure {
            echo 'I will run if failure'
        }
        aborted {
            echo 'pipeline is aborted'
        }
    }
}





