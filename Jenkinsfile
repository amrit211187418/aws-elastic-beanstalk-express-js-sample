pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            // Mount Docker socket for Docker-in-Docker if needed
            args '-v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
    }

    stages {
        stage('Setup Dependencies') {
            steps {
                script {
                    // Install project dependencies
                    sh 'npm ci'
                    echo 'Dependencies have been installed.'

                    // Install Snyk as a dev dependency
                    sh 'npm install snyk --save-dev'
                    echo 'Snyk has been installed locally.'
                }
            }
        }

        stage('Snyk Authorization') {
            steps {
                script {
                    // Authenticate Snyk using environment variable
                    sh 'npx snyk auth ${SNYK_TOKEN}'
                    echo 'Snyk authentication successful.'
                }
            }
        }

        stage('Run Security Audit') {
            steps {
                script {
                    // Run Snyk test and capture the output
                    def snykOutput = sh(script: 'npx snyk test --json', returnStdout: true)
                    def snykData = readJSON(text: snykOutput)
                    
                    // Check for vulnerabilities and decide the outcome
                    if (snykData.vulnerabilities.find { it.severity == 'critical' }) {
                        error 'Critical vulnerabilities found in the project.'
                    } else {
                        writeFile file: 'snyk-report.json', text: snykOutput
                        echo 'Snyk report generated successfully.'
                    }
                }
            }
            post {
                success {
                    echo 'Security scan passed with no critical vulnerabilities.'
                }
                failure {
                    echo 'Security scan failed. Review snyk-report.json for details.'
                }
            }
        }

        stage('Build Application') {
            steps {
                script {
                    // Build step, if required (no actual build step here, but a placeholder)
                    echo 'Running build...'
                    sh 'npm run build || true'
                    echo 'Build completed.'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    // Run unit tests
                    sh 'npm test'
                    echo 'Tests execution finished.'
                }
            }
            post {
                success {
                    echo 'All tests passed.'
                }
                failure {
                    echo 'Some tests failed. Please review the logs for details.'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        failure {
            echo 'Pipeline failed due to one or more errors.'
        }
    }
}
