def registry = 'https://ngiridhar.jfrog.io'

pipeline {
    agent any
    environment {
        MAVEN_HOME = '/opt/maven'
    }

    stages {
        stage("Build") {
            steps {
                echo "--------- Build started ----------"
                withEnv(["PATH=${MAVEN_HOME}/bin:${env.PATH}"]) {
                    sh 'mvn clean deploy -Dmaven.test.skip=true' // Skipping tests during build
                }
                echo "--------- Build completed --------"
            }
        }

        stage("Test") {
            steps {
                echo "---------- Unit tests started -------"
                withEnv(["PATH=${MAVEN_HOME}/bin:${env.PATH}"]) {
                    sh 'mvn test' // Running tests separately
                }
                echo "---------- Unit tests completed -------"
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'saidemy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('saidemy-sonarqube-server') {
                    echo "---------- SonarQube analysis started -------"
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=your_project_key -Dsonar.projectName=your_project_name"
                    echo "---------- SonarQube analysis completed -------"
                }
            }
        }

        stage("Quality Gate") {
            steps {
                script {
                    echo "---------- Checking Quality Gate status -------"
                    timeout(time: 1, unit: 'HOURS') {
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
                    def server = artifactory.server("artifactory-server-name") // Ensure server is defined
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "jarstaging/*.jar",
                                "target": "giridhar-libs-release-local/",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions": ["*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo "-------------- JAR publish completed --------------"
                }
            }
        }
    }
}

