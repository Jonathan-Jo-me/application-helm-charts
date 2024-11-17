pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Kubelinter Helm Charts') {
            steps {
                script {
                    try {
                        // Check if the directory already exists before cloning
                        if (fileExists('application-helm-charts')) {
                            echo "Directory 'application-helm-charts' already exists. Skipping clone."
                        } else {
                            // Clone the repo with the Helm charts if the directory doesn't exist
                            sh 'git clone https://gitlab.com/Jonathan.mee/application-helm-charts.git'
                        }
                        
                        // Navigate into the cloned repository (or existing one)
                        dir('application-helm-charts') {  // Use dir block to change to the correct working directory
                            // Define the list of chart directories
                            def chartDirs = ['database', 'api', 'web']  // Update with actual chart paths

                            // Loop through each chart and run Kubelinter
                            for (chart in chartDirs) {
                                echo "Running Kubelinter on ${chart}..."

                                // Run Kubelinter and capture both the status and the output
                                def lintResult = sh(script: "kube-linter lint ${chart}/ --format=sarif", returnStatus: true, returnStdout: true).trim()

                                // Log the linter output for debugging
                                echo "Kubelinter output for ${chart}: ${lintResult}"

                                // Check the return status of the linter (exit code)
                                if (currentBuild.result == 'FAILURE') {
                                    echo "Kubelinter detected issues in ${chart}, but continuing with the pipeline."
                                } else {
                                    echo "No high-priority issues detected in ${chart}."
                                }
                            }
                        }
                    } catch (Exception e) {
                        error("Error running Kubelinter: ${e.message}")
                    }
                }
            }
        }
    }

    post {
        always {
            // Archive the Kubelinter report for review
            archiveArtifacts artifacts: '*-kubelint-report.sarif', allowEmptyArchive: true
        }
    }
}
