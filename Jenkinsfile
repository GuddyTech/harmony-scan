def ISSUE_TITLE = 'This Example Issue Title'
def ISSUE_BODY = 'This is the body of the example issue.'
def ISSUE_LABELS = '["bug", "help wanted"]'

def GITHUB_REPO = 'guddytech/harmony-scan' // Replace with your GitHub repository
def GITHUB_API_URL = "https://api.github.com/repos/${GITHUB_REPO}"

pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
            }
        }

        stage('Unit Test') {
            steps {
                echo 'Running unit tests...'
            }
        }

        stage('Harmony Scan') {
            steps {
                catchError(message: 'Failed to get Harmony', buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    script {
                        scan = 'vulnerability'
                        
                        def vulnerabilitiesFound = scan.contains('vulnerability')

                        if (vulnerabilitiesFound) {
                            echo 'Vulnerabilities found, creating or updating GitHub issue...'
                            withCredentials([string(credentialsId: 'githubpat-30-05-24-finegrained', variable: 'GITHUB_TOKEN')]) {
                                def issueExists = sh(script: '''
                                    curl -s -L \
                                    -H "Authorization: token ${GITHUB_TOKEN}" \
                                    -H "Accept: application/vnd.github+json" \
                                    ${GITHUB_API_URL}/issues \
                                    | jq -r --arg title "${ISSUE_TITLE}" '.[] | select(.title == $title) | .number'
                                ''', returnStdout: true).trim()
                                
                                if (issueExists) {
                                    def issueNumber = issueExists
                                    echo "Updating existing issue #${issueNumber}..."
                                    def jsonPayload = """
                                    {
                                        "title": "${ISSUE_TITLE}",
                                        "body": "${ISSUE_BODY}",
                                        "labels": ${ISSUE_LABELS}
                                    }
                                    """
                                    echo "JSON Payload: ${jsonPayload}"
                                    sh(script: '''
                                        curl -s -L \
                                        -X PATCH \
                                        -H "Authorization: token ${GITHUB_TOKEN}" \
                                        -H "Accept: application/vnd.github+json" \
                                        ${GITHUB_API_URL}/issues/${issueNumber} \
                                        -d '${jsonPayload}'
                                    ''')
                                } else {
                                    echo "Creating new issue..."
                                    def jsonPayload = """
                                    {
                                        "title": "${ISSUE_TITLE}",
                                        "body": "${ISSUE_BODY}",
                                        "labels": ${ISSUE_LABELS}
                                    }
                                    """
                                    echo "JSON Payload: ${jsonPayload}"
                                    sh(script: '''
                                        curl -s -L \
                                        -X POST \
                                        -H "Authorization: token ${GITHUB_TOKEN}" \
                                        -H "Accept: application/vnd.github+json" \
                                        ${GITHUB_API_URL}/issues \
                                        -d '${jsonPayload}'
                                    ''')
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
                echo 'Deploying...'
            }
        }
    }
}
