pipeline {
    agent {
        docker {
            image 'node:18-alpine'
            reuseNode true
        }
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    ls -l build/index.html
                    CI=true npm test -- --watchAll=false
                '''

                junit 'test-results/junit.xml'
            }
        }
    }
}