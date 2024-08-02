pipeline {

    // Specify the agent to run the pipeline
    agent any
    // Set environment variables for the pipeline
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    // Define the stages of the pipeline
    stages {
        // Stage for building the project
        stage("build") {
            steps {
                // Log message to indicate build start
                echo "----------- build started ----------"
                // Run Maven clean and deploy commands, skipping tests
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                // Log message to indicate build completion
                echo "----------- build completed ----------"
            }
        }

        // Stage for running unit tests
        stage("test") {
            steps {
                // Log message to indicate unit test start
                echo "----------- unit test started ----------"
                // Run Maven Surefire report
                sh 'mvn surefire-report:report'
                // Log message to indicate unit test completion
                echo "----------- unit test completed ----------"
            }
        }

        // Stage for SonarQube analysis
        stage('SonarQube analysis') {
            environment {
                // Set the SonarQube scanner tool
                scannerHome = tool 'saidemy-sonar-scanner'
            }
            steps {
                // Execute SonarQube analysis within the SonarQube environment
                withSonarQubeEnv('saidemy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        // Stage for Quality Gate check
        stage("Quality Gate") {
            steps {
                script {
                    // Set a timeout for the quality gate check
                    timeout(time: 1, unit: 'HOURS') {
                        // Wait for the quality gate result
                        def qg = waitForQualityGate()
                        // Check if the quality gate status is not OK
                        if (qg.status != 'OK') {
                            // Abort the pipeline if quality gate fails
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }
    }
}