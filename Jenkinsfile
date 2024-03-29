#!groovy
/**
IBM KAPAMILYA
GROUP 3

DECLARATIVE PIPELINE

Reference
    * https://github.com/maxyermayank/jenkins-pipeline-demo-pwa/blob/develop/Jenkinsfile
    * https://www.avioconsulting.com/blog/using-role-based-authorization-strategy-jenkins
*/

def USER_INPUT_DEYPLOY_TO_STAGING

pipeline{
    agent any

    options {
        timestamps()
        //timeout time:10, unit:'MINUTES'
    }

    parameters {
        string(defaultValue: "develop", description: 'Branch Specifier', name: 'GIT_SPECIFIER')
        string(defaultValue: "https://github.com/detorresrc/devops-angular7", description: 'GIT URL', name: 'GIT_URL')
    }

    stages {
        stage("Initialize") {
            steps {
                script {
                    notifyBuild('STARTED')
                    echo "${BUILD_NUMBER} - ${env.BUILD_ID} on ${env.JENKINS_URL}"
                    sh 'rm -rf dist/'
                }
            }
        }
        stage('Source') {
            steps {
                git branch: "${params.GIT_SPECIFIER}", url: "${params.GIT_URL}"
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Test - Run the linter (tslint)') {
            steps {
                sh 'npm run lint'
            }
        }
        stage('Sonarqube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'SONARQUBE_APIKEY', variable: 'SONARQUBE_APIKEY')]) {
                    withSonarQubeEnv('SonarqubeServer_GROUP3') {
                        sh '/usr/local/bin/sonar-scanner -Dsonar.login=$SONARQUBE_APIKEY -Dsonar.projectVersion=$BUILD_NUMBER'
                    }
                }
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                } 
            }
        }
        stage('Build') {
            steps {
                sh 'npm run build:prod:en'
            }
        }
        stage('Upload Artifact') {
            steps {
                withCredentials([string(credentialsId: 'ARTIFACTORY_USERNAME', variable: 'ARTIFACTORY_USERNAME'), string(credentialsId: 'ARTIFACTORY_PASSWORD', variable: 'ARTIFACTORY_PASSWORD')]) {
                    sh '''
                        ARTIFACT_NAME=artifact.group3.$BUILD_NUMBER.tar.bz2
                        tar -cj dist/* > $ARTIFACT_NAME
                        curl -u$ARTIFACTORY_USERNAME:$ARTIFACTORY_PASSWORD -T $ARTIFACT_NAME "https://jfrog.ibm-kapamilya-devops.com/artifactory/generic-local/$ARTIFACT_NAME"
                    '''
                }
            }
        }
        stage('Deploy - Staging') {
            steps {
/*
                script {
                    timeout(time: 7, unit: 'DAYS') {
                        userInputDeployToStaging = input(
                            id: 'USER_INPUT_DEYPLOY_TO_STAGING',
                            message: 'Deploy to staging server?', 
                            submitter: 'abscbncorpdevelopment',
                            parameters: [
                            ]
                        )
                    }
                }
*/
            }
        }
    }

    post {
        /*
            * These steps will run at the end of the pipeline based on the condition.
            * Post conditions run in order regardless of their place in pipeline
            * 1. always - always run
            * 2. changed - run if something changed from last run
            * 3. aborted, success, unstable or failure - depending on status
            */
        always {
            echo "I AM ALWAYS first"
            notifyBuild("${currentBuild.currentResult}")
        }
        aborted {
            echo "BUILD ABORTED"
        }
        success {
            echo "BUILD SUCCESS"
        }
        unstable {
            echo "BUILD UNSTABLE"
        }
        failure {
            echo "BUILD FAILURE"
        }
    }
}

def getCurrentBranch () {
    return sh (
            script: 'git rev-parse --abbrev-ref HEAD',
            returnStdout: true
    ).trim()
}

def getShortCommitHash() {
    return sh(returnStdout: true, script: "git log -n 1 --pretty=format:'%h'").trim()
}

def getChangeAuthorName() {
    return sh(returnStdout: true, script: "git show -s --pretty=%an").trim()
}

def getChangeAuthorEmail() {
    return sh(returnStdout: true, script: "git show -s --pretty=%ae").trim()
}

def getChangeSet() {
    return sh(returnStdout: true, script: 'git diff-tree --no-commit-id --name-status -r HEAD').trim()
}

def getChangeLog() {
    return sh(returnStdout: true, script: "git log --date=short --pretty=format:'%ad %aN <%ae> %n%n%x09* %s%d%n%b'").trim()
}

def notifyBuild(String buildStatus = 'STARTED') {
    // build status of null means successful
    buildStatus = buildStatus ?: 'SUCCESS'

    def branchName = getCurrentBranch()
    def shortCommitHash = getShortCommitHash()
    def changeAuthorName = getChangeAuthorName()
    def changeAuthorEmail = getChangeAuthorEmail()
    def changeSet = getChangeSet()
    def changeLog = getChangeLog()

    // Default values
    def colorName = 'RED'
    def colorCode = '#FF0000'
    def subject = "${buildStatus}: '${env.JOB_NAME} [${env.BUILD_NUMBER}]'" + branchName + ", " + shortCommitHash
    def summary = "Started: Name:: ${env.JOB_NAME} \n " +
            "Build Number: ${env.BUILD_NUMBER} \n " +
            "Build URL: ${env.BUILD_URL} \n " +
            "Short Commit Hash: " + shortCommitHash + " \n " +
            "Branch Name: " + branchName + " \n " +
            "Change Author: " + changeAuthorName + " \n " +
            "Change Author Email: " + changeAuthorEmail + " \n " +
            "Change Set: " + changeSet

    if (buildStatus == 'STARTED') {
        color = 'YELLOW'
        colorCode = '#FFFF00'
    } else if (buildStatus == 'SUCCESS') {
        color = 'GREEN'
        colorCode = '#00FF00'
    } else {
        color = 'RED'
        colorCode = '#FF0000'
    }

    println "${summary}"
}
