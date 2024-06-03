//It works but creates the issue any time the pipeline runs. Check github-issue/RestAPI-working-updates-issues


def ISSUE_TITLE = "This is for Harmony Scan Example Issue Title. REPO: $JOB_NAME BUILD NUMBER: $BUILD_DISPLAY_NAME" 
def ISSUE_BODY = "This is the body of the example issue issue. Details: ${scan}"
def ISSUE_LABELS = '["bug", "help wanted"]'

def GITHUB_REPO = 'guddytech/harmony-scan'; // Replace with your GitHub repository
def GITHUB_API_URL = "https://api.github.com/repos/${GITHUB_REPO}/issues";

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
                            echo 'Vulnerabilities found, creating GitHub issue...'
                            // def issueTitle = 'Vulnerabilities found in Harmony scan'
                            // def issueBody = "Harmony scan detected vulnerabilities in the codebase. Details:\n\n${scan}"
                            // def issueLabels = '["bug", "help wanted"]'                         

                            withCredentials([string(credentialsId: 'githubpat-30-05-24-finegrained', variable: 'GITHUB_TOKEN')]) {
                                // Create GitHub issue
                                def jsonPayload = """
                                {
                                    "title": "${ISSUE_TITLE}",
                                    "body": "${ISSUE_BODY}",
                                    "labels": ${ISSUE_LABELS}
                                }
                                """
                                sh '''#!/bin/bash
                                    curl -s -L \
                                    -X POST \
                                    -H "Authorization: token ${GITHUB_TOKEN}" \
                                    -H "Accept: application/vnd.github+json" \
                                    -H "X-GitHub-Api-Version: 2022-11-28" \
                                    ''' + GITHUB_API_URL + ''' \
                                    -d ''' + "'" + jsonPayload + "'"                                   
                                
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
