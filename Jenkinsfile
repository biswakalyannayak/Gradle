def ARTIFACT = ''
def VERSION = ''
def CONTAINER = ''
pipeline {
    agent any

    environment {
        CONT_HOME = '/tmp/home'

        GRADLE = "./gradlew -Duser.home=$CONT_HOME"

        VERSION =  node('master') { sh(script: " $GRADLE properties | grep \'version:\'  | awk \'{print \$2}\'", returnStdout: true).trim()}

    }

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Set Variable') {
            agent any
            steps {
                echo "A-found value  ${ARTIFACT}"
                echo "cont home ${CONT_HOME}"
                echo "gradle home ${GRADLE}"
                echo "V-found value  ${VERSION}"
                script{
                     ARTIFACT =  sh ( script: "gradle properties | grep \'group:\'  | awk \'{print \$2}\'",
                                   returnStdout: true
                                 ).trim()
                     VERSION =  sh ( script: "gradle properties | grep \'version:\'  | awk \'{print \$2}\'",
                                  returnStdout: true
                                ).trim()
                     CONTAINER = "biswakalyan/${ARTIFACT}-${GIT_BRANCH}-${VERSION}-${currentBuild.startTimeInMillis}-${GIT_COMMIT}"
                }
                echo "A-found value after ${ARTIFACT}"
                echo "V-found value after ${VERSION}"
                echo "C-found value after ${CONTAINER}"
            }
        }
        stage('Print variables') {
            agent any
            steps {
                script{
                    CONTAINER = "biswakalyan/${ARTIFACT}-${GIT_BRANCH}-${VERSION}-${currentBuild.startTimeInMillis}-${GIT_COMMIT}"
                }
                sh "printenv | sort"
            }
        }
        stage('Build') {
            agent any
            steps {
                checkout scm
                withGradle() {
                    sh 'gradle clean build --rerun-tasks'
                }
            }

            post {
                success {
                    junit '**/build/test-results/test/TEST-*.xml'
                    archiveArtifacts '**/libs/*.jar'
                }
            }
        }

        stage('Integration Test') {
            agent any
            steps {
                echo "run integration tests"
                 withGradle() {
                   sh 'gradle integrationTest'
                }

            }
             post {
                success {
                    junit '**/build/test-results/integrationTest/TEST-*.xml'
                }
            }
        }

        stage('Sonarqube') {
            agent any
            steps {
                withSonarQubeEnv('code-quality'){
                    withGradle(){
                         sh 'gradle --info sonarqube'
                    }
                }
            }
        }

        stage('Push to registry') {
            agent any
            steps {
                timeout(time: 60, unit: 'SECONDS') {
                    script {
                        env.QA = input message: 'Publish to QA?', ok: 'Pass!', submitter: 'GRP-QA-DEPLOYER', submitterParameter: "submitter"
                    }
                    echo "QA publish!"
                }
                sh "printenv | sort"
            }
        }
        stage('Peer review') {
            agent any
            steps {
                timeout(time: 10, unit: 'DAYS') {
                    script {
                        env.SIT = input message: 'Is peer review pass?', ok: 'Pass!', submitter: 'GRP-PEER-REVIEW', submitterParameter: "submitter"
                    }
                    echo "Peer review passed!"
                }
                sh "printenv | sort"
            }
        }

    }
}