pipeline {
    agent none

    options {
        skipDefaultCheckout(true)
    }

    environment {
        APP_NAME = 'tp1-devops'
        SONAR_PROJECT_KEY = 'lucas-riyad'
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
            }
        }
        stage('SonarQube Analysis') {
            agent any
            environment {
                SCANNER_HOME = tool 'sonar-scanner'
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
    }
}