def ISSUE_TITLE = "This is for Harmony Scan Example Issue Title. REPO: $JOB_NAME BUILD NUMBER: $BUILD_DISPLAY_NAME"
def ISSUE_BODY = "This is the body of the example issue issue."
def ISSUE_LABELS = '["bug", "help wanted"]'

def GITHUB_REPO = 'guddytech/harmony-scan' // Replace with your GitHub repository
def GITHUB_API_URL = "https://api.github.com/repos/${GITHUB_REPO}/issues"

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                // Your build steps here
                echo 'Building...'
            }
        }

        stage('Unit Test') {
            steps {
                // Your unit test steps here
                echo 'Running unit tests...'
            }
        }

        stage('Harmony Scan') {
            steps {
                catchError(message: 'Failed to get Harmony', buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    script {
                        // scan = new ors.security.CommonHarmony(steps, env, Artifactory, scm).run_scan([
                        //     "repository":"mint/araas",
                        //     "product_output":"${env.WORKSPACE}",
                        //     "analyze_results": true, 
                        //     "exclusions": [                          // Add exclusions here
                        //     "**/node_modules/**",               // Exclude node_modules directory
                        //     "**/*.log"                         // Exclude log files
                        //     ]
                        // ])

                        scan = 'vulnerability'
                        
                        // Assuming scan results are stored in scan variable
                        def vulnerabilitiesFound = scan.contains('vulnerability') // Adjust based on actual scan output format

                        if (vulnerabilitiesFound) {
                            echo 'Vulnerabilities found, checking for existing GitHub issue...'

                            withCredentials([string(credentialsId: 'githubpat-30-05-24-finegrained', variable: 'GITHUB_TOKEN')]) {
                                def issueNumber = getExistingIssueNumber(ISSUE_TITLE)

                                if (issueNumber) {
                                    echo "Updating existing issue #${issueNumber}..."
                                    updateIssue(issueNumber, ISSUE_TITLE, ISSUE_BODY, ISSUE_LABELS)
                                } else {
                                    echo 'Creating new GitHub issue...'
                                    createIssue(ISSUE_TITLE, ISSUE_BODY, ISSUE_LABELS)
                                }
                            }
                        } else {
                            echo 'No vulnerabilities found.'
                        }
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                // Your deploy steps here
                echo 'Deploying...'
            }
        }
    }
}

def getExistingIssueNumber(title) {
    def response = sh(script: """
        curl -s -H "Authorization: token ${GITHUB_TOKEN}" \
             -H "Accept: application/vnd.github+json" \
             -H "X-GitHub-Api-Version: 2022-11-28" \
             ${GITHUB_API_URL}?state=open | jq '.[] | select(.title=="${title}") | .number'
    """, returnStdout: true).trim()

    return response ? response.toInteger() : null
}

def updateIssue(issueNumber, title, body, labels) {
    def jsonPayload = """
    {
        "title": "${title}",
        "body": "${body}",
        "labels": ${labels}
    }
    """
    sh(script: """
        curl -s -L -X PATCH \
             -H "Authorization: token ${GITHUB_TOKEN}" \
             -H "Accept: application/vnd.github+json" \
             -H "X-GitHub-Api-Version: 2022-11-28" \
             ${GITHUB_API_URL}/${issueNumber} \
             -d '${jsonPayload}'
    """)
}

def createIssue(title, body, labels) {
    def jsonPayload = """
    {
        "title": "${title}",
        "body": "${body}",
        "labels": ${labels}
    }
    """
    sh(script: """
        curl -s -L -X POST \
             -H "Authorization: token ${GITHUB_TOKEN}" \
             -H "Accept: application/vnd.github+json" \
             -H "X-GitHub-Api-Version: 2022-11-28" \
             ${GITHUB_API_URL} \
             -d '${jsonPayload}'
    """)
}
