//This one works! It creates issues if they are not already there, and updates the existing issue if it is there. 
//It works with the issue title

def GITHUB_REPO = 'guddytech/harmony-scan'; // Replace with your GitHub repository
def GITHUB_API_URL = "https://api.github.com/repos/${GITHUB_REPO}";
def ISSUES_URL = "https://github.com/${GITHUB_REPO}/issues/52"; //Issues URL for just the harmony scan that keeps updating that number 52 when it sees a vulnerability
//def BUILD_URL = "${env.JENKINS_URL}job/${env.JOB_NAME}/${env.BUILD_NUMBER}/"; //build url
//def BUILD_URL = "${env.JENKINS_URL}job/${env.JOB_NAME.replaceAll(' ', '%20')}/job/${env.BUILD_NUMBER}/";

pipeline {
    agent any 

    stages {
        stage('Build') {
            steps {
                // Your build steps here
                echo 'Building...'
                echo  "${ISSUES_URL}"
                echo "${BUILD_URL}"
                echo "${JENKINS_URL}"
                echo "${JOB_NAME}"
                echo "${BUILD_NUMBER}"
                echo "${JOB_NAME}"
                echo "${JOB_NAME.replaceAll(' ', '%20')}"
                echo "${env.BUILD_URL}"
                
                                
            }
        }

        stage('Unit Test') {
            steps {
                // Your unit test steps here
                echo 'Running unit tests...'
                //def encodedJobName = java.net.URLEncoder.encode(env.JOB_NAME, "UTF-8")
                //def urlFragment = "${encodedJobName}-#${env.BUILD_NUMBER}"
                //echo "Job URL Fragment: ${urlFragment}"
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

                        //def encodedJobName = java.net.URLEncoder.encode(env.JOB_NAME, "UTF-8")
                        //def urlFragment = "${encodedJobName}-#${env.BUILD_NUMBER}"
                        //echo "Job URL Fragment: ${urlFragment}"
                        echo  "${ISSUES_URL}"

                        scan = 'vulnerability'
                        
                        // Assuming scan results are stored in scan variable
                        def vulnerabilitiesFound = scan.contains('vulnerability') // Adjust based on actual scan output format

                        if (vulnerabilitiesFound) {
                            echo 'Vulnerabilities found, creating or updating GitHub issue...'

                            def dateFormat = new java.text.SimpleDateFormat("yyyy-MM-dd HH:mm:ss")
                            def currentDateTime = dateFormat.format(new Date())

                            //def ISSUE_TITLE = "Test for Harmony Scan BlackDuck. BUILD NUMBER: $BUILD_DISPLAY_NAME"
                            def ISSUE_TITLE = "Test for Harmony Scan BlackDuck."
                            def ISSUE_BODY = "This is the body of the example issuesss arising now. \\nThe Issue URL: ${ISSUES_URL} \\nBuild_Url: ${BUILD_URL} \\nDate and Time: ${currentDateTime} \\nDetails: ${scan}"
                            def ISSUE_LABELS = '["bug", "help wanted"]'


                            withCredentials([string(credentialsId: 'githubpat-30-05-24-finegrained', variable: 'GITHUB_TOKEN')]) {
                                // Check if an issue with the same title already exists
                                def issueExists = sh(
                                    script: '''#!/bin/bash
                                    curl -s -L \
                                    -H "Authorization: token $GITHUB_TOKEN" \
                                    -H "Accept: application/vnd.github+json" \
                                    ''' + GITHUB_API_URL + '''/issues \
                                    | jq -r --arg title "''' + ISSUE_TITLE + '''" '.[] | select(.title == $title) | .number'
                                    ''',
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
                                    sh '''#!/bin/bash
                                        curl -s -L \
                                        -X PATCH \
                                        -H "Authorization: token $GITHUB_TOKEN" \
                                        -H "Accept: application/vnd.github+json" \
                                        ''' + GITHUB_API_URL + '''/issues/''' + issueNumber + ''' \
                                        -d ''' + "'" + jsonPayload + "'"
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
                                    sh '''#!/bin/bash
                                        curl -s -L \
                                        -X POST \
                                        -H "Authorization: token $GITHUB_TOKEN" \
                                        -H "Accept: application/vnd.github+json" \
                                        ''' + GITHUB_API_URL + '''/issues \
                                        -d ''' + "'" + jsonPayload + "'"
                                }
                            }
                        } else {
                            echo 'No vulnerabilities found.'
                            echo  "${ISSUES_URL}"
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
