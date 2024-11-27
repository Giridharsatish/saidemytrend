pipeline {
    agent any
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }

    stages {
        stage("build") {
            steps {
                echo "--------- build started ----------"
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo "--------- build completed --------"
            }
        }

        stage('test') {
            steps {
                echo "---------- unit test started -------"
                sh 'mvn test' // This will actually run the tests
                echo "---------- unit test completed -------"
            }
        }

        stage('SonarQube analysis') {
            environment {
                scannerHome = tool 'saidemy-sonar-scanner'
            }
            steps {
                withSonarQubeEnv('saidemy-sonarqube-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage("Quality gate") {
            steps {
                script {
                    timeout(time: 1, unit: 'HOURS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }
                }
            }
        }

        stage('jar publish') {
            steps {
                script {
                    echo "------------ jar publish started ---------------"
                    def server = artifactory.server("artifactory-server-name") // Correct server config
                    def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}"
                    def uploadSpec = """{
                        "files":  [
                            {
                                "pattern": "jarstaging/(*)",
                                "target": "giridhar-libs-release-local/{1}",
                                "flat": "false",
                                "props": "${properties}",
                                "exclusions" : [ "*.sha1", "*.md5"]
                            }
                        ]
                    }"""
                    def buildInfo = server.upload(uploadSpec)
                    buildInfo.env.collect()
                    server.publishBuildInfo(buildInfo)
                    echo "-------------- jar publish ended --------------"
                }
            }
        }
    }
}

