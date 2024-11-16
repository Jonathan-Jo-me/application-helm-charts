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
                        // Define the list of chart directories
                        def chartDirs = ['database', 'api', 'web'] // Update with actual chart paths

                        // Loop through each chart and run Kubelinter
                        for (chart in chartDirs) {
                            echo "Running Kubelinter on ${chart}..."
                            sh "kube-linter lint ${chart}/ --format=sarif > ${chart}-kubelint-report.sarif"
                            
                            // Check for high-priority issues in the generated report
                            def result = sh(script: "grep -q 'severity: \"high\"' ${chart}-kubelint-report.sarif", returnStatus: true)
                            if (result == 0) {
                                error("High-priority issues detected in ${chart} by Kubelinter.")
                            } else {
                                echo "No high-priority issues detected in ${chart}."
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
