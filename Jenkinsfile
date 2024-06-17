pipeline {
    agent any 

    environment {
        // Define any environment variables you need
        WORKSPACE = "${env.WORKSPACE}"  
    }

    stages {
        stage('Harmony Scan') {
            steps {
                catchError(message: 'Failed to get Harmony', buildResult: 'UNSTABLE', stageResult: 'UNSTABLE') {
                    script {
                        try {
                            // Download and configure OWASP Dependency-Check
                            //sh 'curl -L https://github.com/jeremylong/DependencyCheck/releases/download/v6.3.2/dependency-check-6.3.2-release.zip -o dependency-check.zip'
                           // sh 'unzip dependency-check.zip -d dependency-check'
                            
                            // Run Dependency-Check scan with exclusions
                            sh '''
                                ./dependency-check/bin/dependency-check.sh --project "my-project" \
                                --scan ${WORKSPACE}/**/*.jar \
                                --out ${WORKSPACE}/dependency-check-report \
                                --format ALL \
                                --exclude "**/node_modules/**,**/*.log"
                            '''

                            // Process scan results
                            scanResults = readFile("${WORKSPACE}/dependency-check-report/dependency-check-report.json")
                            def scanResultsJson = new groovy.json.JsonSlurper().parseText(scanResults)
                            
                            // Check if dependencies exist and for vulnerabilities
                            if (scanResultsJson.dependencies) {
                                def vulnerabilities = scanResultsJson.dependencies.findAll { it.vulnerabilities?.size() > 0 }
                                if (vulnerabilities.size() > 0) {
                                    currentBuild.result = 'UNSTABLE'
                                    echo "Vulnerabilities found: ${vulnerabilities.size()}"
                                } else {
                                    echo "No vulnerabilities found"
                                }
                            } else {
                                echo "No dependencies found"
                                currentBuild.result = 'UNSTABLE'
                            }

                        } catch (Exception e) {
                            echo "Scan failed: ${e.message}"
                            currentBuild.result = 'UNSTABLE'
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'dependency-check-report/*', allowEmptyArchive: true
            publishHTML(target: [
                reportDir: 'dependency-check-report',
                reportFiles: 'dependency-check-report.html',
                reportName: 'Dependency Check Report'
            ])
        }
    }
}
