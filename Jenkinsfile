def ISSUE_TITLE = 'this'
def ISSUE_BODY = 'This is the body of the example issue.'
def ISSUE_LABELS = '["bug", "help wanted"]'

def GITHUB_REPO = 'guddytech/harmony-scan'; // Replace with your GitHub repository
def GITHUB_API_URL = "https://api.github.com/repos/${GITHUB_REPO}";

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
                            echo 'Vulnerabilities found, creating or updating GitHub issue...'
                            withCredentials([string(credentialsId: 'githubpat-30-05-24-finegrained', variable: 'GITHUB_TOKEN')]) {
                                // Check if an issue with the same title already exists
                                def issueExists = sh(
                                    script: """
                                    curl -s -L \
                                    -H "Authorization: token ${GITHUB_TOKEN}" \
                                    -H "Accept: application/vnd.github+json" \
                                    ${GITHUB_API_URL}/issues \
                                    | jq -r --arg title "${ISSUE_TITLE}" '.[] | select(.title == \$title) | .number'
                                    """,
                                    returnStdout: true
                                ).trim()
                                
                                if (issueExists) {
                                    // Update the existing issue
                                    def issueNumber = issueExists
                                    echo "Updating existing issue #${issueNumber}..."
                                    def jsonPayload = """
                                    {
                                        "title": "${ISSUE_TITLE}",
                                        "body": "${ISSUE_BODY}",
                                        "labels": ${ISSUE_LABELS}
                                    }
                                    """
                                    sh """
                                        curl -s -L \
                                        -X PATCH \
                                        -H "Authorization: token ${GITHUB_TOKEN}" \
                                        -H "Accept: application/vnd.github+json" \
                                        ${GITHUB_API_URL}/issues/${issueNumber} \
                                        -d '${jsonPayload}'
                                    """
                                } else {
                                    // Create a new issue
                                    echo "Creating new issue..."
                                    def jsonPayload = """
                                    {
                                        "title": "${ISSUE_TITLE}",
                                        "body": "${ISSUE_BODY}",
                                        "labels": ${ISSUE_LABELS}
                                    }
                                    """
                                    sh """
                                        curl -s -L \
                                        -X POST \
                                        -H "Authorization: token ${GITHUB_TOKEN}" \
                                        -H "Accept: application/vnd.github+json" \
                                        ${GITHUB_API_URL}/issues \
                                        -d '${jsonPayload}'
                                    """
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
