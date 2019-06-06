/**
IBM KAPAMILYA
GROUP 3

DECLARATIVE PIPELINE
*/
pipeline{
    agent any
    stages {
        stage('Source') {
            steps {
                git branch: 'develop', url: 'https://github.com/detorresrc/devops-angular7'
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
        stage('Upload') {
            steps {
                withCredentials([string(credentialsId: 'ARTIFACTORY_USERNAME', variable: 'ARTIFACTORY_USERNAME'), string(credentialsId: 'ARTIFACTORY_PASSWORD', variable: 'ARTIFACTORY_PASSWORD')]) {
                    sh '''
                        tar -cj dist/* > artifact.$BUILD_NUMBER.tar.bz2
                        curl -u$ARTIFACTORY_USERNAME:$ARTIFACTORY_PASSWORD -T artifact.$BUILD_NUMBER.tar.bz2 "https://jfrog.ibm-kapamilya-devops.com/artifactory/generic-local/artifact.$BUILD_NUMBER.tar.bz2"
                    '''
                }
            }
        }
    }
}