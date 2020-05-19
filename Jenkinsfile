pipeline {
    agent any

    tools {
        gradle 'gradle-6.4'
    }

    environment {
        ARTIFACT =  sh (script: "gradle properties | grep \'group:\'  | awk \'{print \$2}\'",
                                          returnStdout: true
                                        ).trim()
        VERSION =  sh ( script: " gradle properties | grep \'version:\'  | awk \'{print \$2}\'",
                        returnStdout: true
                      ).trim()
        CONTAINER = "biswakalyan/${ARTIFACT}-${GIT_BRANCH}-${VERSION}-${currentBuild.startTimeInMillis}-${GIT_COMMIT}"
        IMAGE = "biswakalyan/${ARTIFACT}:${VERSION}-${BUILD_NUMBER}"
        GIT_RELEASE_TAG = "release/${ARTIFACT}@${VERSION}-${BUILD_NUMBER}"
    }
    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Print variables') {
            steps {
                sh "printenv | sort"
            }
        }
        stage('Build') {
            steps {
                checkout scm
                withGradle() {
                    sh 'gradle build --rerun-tasks'
                }
            }

            post {
                success {
                    junit '**/build/reports/jacoco/test/*.xml'
                    archiveArtifacts '**/libs/*.jar'
                }
            }
        }

        stage('Integration Test') {
            steps {
                echo "run integration tests"
                sh 'gradle integrationTest --warning-mode=all'
            }
             post {
                success {
                    junit 'build/reports/**/integrationTest/index.html'
                }
            }
        }

        stage('Sonarqube') {
            steps {
                echo "run sonar"
            }
        }

        stage('Push to registry') {
            steps {
                 echo "run registry"
            }
        }
    }
}