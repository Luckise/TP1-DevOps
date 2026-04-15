pipeline {
    agent none

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            agent any
            steps {
                checkout scm
                stash name: 'source', includes: '**/*'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'oven/bun:latest'
                }
            }
            steps {
                unstash 'source'
                sh 'bun install'
                sh 'bun run build'
                sh 'tar -czf artifact-${BUILD_NUMBER}.tgz -C dist .'
                stash name: 'build-artifact', includes: 'artifact-*.tgz'
            }
        }
        stage('SonarQube Analysis') {
            agent any
            environment {
                SCANNER_HOME = tool 'sonar-scanner'
                SONAR_PROJECT_KEY = 'lucas-riyad'
            }
            steps {
                unstash 'source'
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                          -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                          -Dsonar.projectName="TP1-DevOps" \
                          -Dsonar.sources=src \
                          -Dsonar.exclusions=dist/**,node_modules/**
                    '''
                }
            }
        }

        stage('SonarQube Quality Gate') {
            agent none
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload Artifact to Nexus') {
            agent any
            environment {
                APP_NAME = 'tp1-devops'
                NEXUS_URL = 'http://nexus:8081'
                NEXUS_REPOSITORY = 'ci-raw'
            }
            steps {
                unstash 'build-artifact'
                withCredentials([usernamePassword(credentialsId: 'nexus-credentials', usernameVariable: 'NEXUS_USER', passwordVariable: 'NEXUS_PASS')]) {
                    sh '''
                        curl -f -u "$NEXUS_USER:$NEXUS_PASS" \
                          --upload-file "artifact-${BUILD_NUMBER}.tgz" \
                          "$NEXUS_URL/repository/$NEXUS_REPOSITORY/$APP_NAME/${BUILD_NUMBER}/artifact-${BUILD_NUMBER}.tgz"
                    '''
                }
            }
        }
    }
}