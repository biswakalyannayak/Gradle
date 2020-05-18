pipeline {
    agent any

    tools {
        gradle 'gradle-6.4'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Setup') {
            steps {
            Environment{
                ARTIFACT =  sh (script: "gradle properties | grep 'group='",
                                  returnStdout: true
                                ).trim()
                VERSION =  sh ( script: "gradle properties | grep 'version='",
                                returnStdout: true
                              ).trim()
                CONTAINER = "biswakalyan/${ARTIFACT}-${GIT_BRANCH}-${VERSION}-${currentBuild.startTimeInMillis}-${GIT_COMMIT}"
                IMAGE = "biswakalyan/${ARTIFACT}:${VERSION}-${BUILD_NUMBER}"
                GIT_RELEASE_TAG = "release/${ARTIFACT}@${VERSION}-${BUILD_NUMBER}"
            }
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
                    archiveArtifacts 'build/libs/*.jar'
                }
            }
        }

        stage('Integration Test') {
            steps {
                echo "run integration tests"
                sh 'gradle integrationTest --warning-mode=all'
            }
        }

        stage('Sonarqube') {
            steps {
                echo "run sonar"
            }
        }

        stage('Push to registry') {
            steps {
                 echo "run sonar"
            }
        }
    }
}