def registry = 'https://ngiridhar.jfrog.io'

pipeline {
    agent any
    environment {
        MAVEN_HOME = '/opt/maven' // Define Maven Home
    }

    stages {
        stage("Build") {
            steps {
                echo "--------- Build started ----------"
                withEnv(["PATH=${MAVEN_HOME}/bin:${env.PATH}"]) {
                    // Run Maven clean and deploy, skipping tests during the build phase
                    sh 'mvn clean deploy -Dmaven.test.skip=true'
                }
                echo "--------- Build completed --------"
            }
        }

        stage("Test") {
            steps {
                echo "---------- Unit tests started -------"
                withEnv(["PATH=${MAVEN_HOME}/bin:${env.PATH}"]) {
                    // Run unit tests
                    sh 'mvn test'
                }
                echo "---------- Unit tests completed -------"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'saidemy-sonar-scanner' // Ensure that 'saidemy-sonar-scanner' is correctly installed as a tool
            }
            steps {
                withSonarQubeEnv('saidemy-sonarqube-server') { // Ensure the SonarQube server is correctly configured in Jenkins
                    echo "---------- SonarQube analysis started -------"
                    // Run the SonarQube scanner with the project key and name
                    sh """
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=your_project_key \
                        -Dsonar.projectName=your_project_name
                    """
                    echo "---------- SonarQube analysis completed -------"
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    echo "---------- Checking Quality Gate status -------"
                    timeout(time: 1, unit: 'HOURS') {
                        // Wait for the quality gate result
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                    echo "---------- Quality Gate passed -------"
                }
            }
        }

        stage("Publish JAR to Artifactory") {
            steps {
                script {
                    echo "------------ Publishing JAR started ---------------"
                    // Define the Artifactory server
                    def server = artifactory.server("artifactory-server-name") // Ensure this Artifactory server is configured in Jenkins
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    
                    // Define the upload specification for Artifactory
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "target/*.jar", // Ensure this matches the correct path for the JAR
                                "target": "giridhar-libs-release-local/",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    
                    // Upload the artifact to Artifactory
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect() // Collect environment variables for build information
                    server.publishBuildInfo(buildInfo) // Publish build information to Artifactory
                    echo "-------------- JAR publish completed --------------"
                }
            }
        }
    }
}

