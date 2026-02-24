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
    }
}