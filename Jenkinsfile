pipeline {
    agent {
        docker {
            image 'oven/bun:latest'
        }
    }
    stages {
        stage("Build"){
            steps {
                sh 'bun install'
                sh 'bun run build'
            }
        }
        stage("SonarQube") {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:latest'
                }
            }
            environment {
                SONAR_HOST_URL = 'http://sonarqube:9000'
                SONAR_PROJECT_KEY = 'lucas-riyad-tp1-devops'
            }
            steps {
                withCredentials([string(credentialsId: 'sonar', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=$SONAR_PROJECT_KEY \
                          -Dsonar.projectName=TP1-DevOps \
                          -Dsonar.sources=src \
                          -Dsonar.host.url=$SONAR_HOST_URL \
                          -Dsonar.token=$SONAR_TOKEN
                    '''
                }
            }
        }
    }
}