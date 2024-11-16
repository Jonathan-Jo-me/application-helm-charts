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
                            def lintResult = sh(script: "kube-linter lint ${chart}/ --format=sarif > ${chart}-kubelint-report.sarif", returnStatus: true)
                            
                            // Check if the linter returned an error but continue
                            if (lintResult != 0) {
                                echo "Kubelinter detected issues in ${chart}, but continuing with the pipeline."
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
