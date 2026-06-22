pipeline {
    agent any

    tools {
        maven "Maven3"
        jdk "jdk21"
    }

    environment {
        NEXUS_IP       = '13.126.181.180'
        NEXUS_PORT     = '8081'

        RELEASE_REPO   = 'maven-releases'
        SNAP_REPO      = 'maven-snapshots'
        NEXUS_GRP_REPO = 'maven-public'

        NEXUS_LOGIN    = 'nexuslogin'

        SONARSERVER    = 'sonarserver'
        SONARSCANNER   = 'sonarscanner'
    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn -s settings.xml clean install -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }

            steps {
                withSonarQubeEnv("${SONARSERVER}") {

                    sh """
                    ${scannerHome}/bin/sonar-scanner \
                    -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.sources=src \
                    -Dsonar.java.binaries=target/classes
                    """
                }
            }
        }

        stage('Upload Artifact') {
            steps {

                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUS_IP}:${NEXUS_PORT}",
                    groupId: 'QA',
                    version: "${BUILD_NUMBER}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUS_LOGIN}",

                    artifacts: [
                        [
                            artifactId: 'vprofile',
                            classifier: '',
                            file: 'target/vprofile-v2.war',
                            type: 'war'
                        ]
                    ]
                )
            }
        }
    }

    post {

        success {
            echo 'Pipeline Completed Successfully'
        }

        failure {
            echo 'Pipeline Failed'
        }
    }
}
