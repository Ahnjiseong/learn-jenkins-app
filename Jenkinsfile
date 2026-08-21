pipeline {
    // 전역 에이전트를 사용하지 않음으로써 컨테이너 충돌 방지
    agent none

    environment {
        NETLIFY_SITE_ID = 'e48dccbf-de88-44c2-9340-b05647ea087a'
        NETLIFY_AUTH_TOKEN = credentials('netlify-token')
    }

    stages {
        stage('AWS') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    // aws-cli 이미지는 기본적으로 실행 후 바로 종료되므로 엔트리포인트 무력화
                    args '--entrypoint=""'
                }
            }

            steps {
                sh '''
                    aws --version
                '''
                sh 'aws --version'
            }
        }

        stage('Build') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                }
            }

            steps {
                sh '''
                    echo '빌드 시작..'
                    node --version
                    npm --version
                    npm ci
                    npm run build
                '''
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                }
            }

            steps {
                sh '''
                    npm test
                '''
            }
        }

        stage('E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                }
            }

            steps {
                sh '''
                    # serve를 로컬에 설치하여 실행
                    npm install serve
                    node_modules/.bin/serve -s build & sleep 10
                    npx playwright test --reporter=html
                '''
            }
        }

        stage('Deploy staging') {
            agent {
                docker {
                    image 'node:18-bullseye'
                }
            }

            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify deploy --dir=build
                '''
            }
        }

        stage('Approval') {
            agent none

            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: '운영환경에 배포할까요?', ok: '네 배포합니다'
                }
            }
        }

        stage('Deploy prod') {
            agent {
                docker {
                    image 'node:18-bullseye'
                }
            }

            steps {
                sh '''
                    npm install netlify-cli@20.1.1
                    node_modules/.bin/netlify deploy --dir=build --prod
                '''
            }
        }

        stage('Prod E2E') {
            agent {
                docker {
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                }
            }

            environment {
                CI_ENVIRONMENT_URL = 'https://verdant-macaron-8d142a.netlify.app'
            }

            steps {
                sh '''
                    echo "================================="
                    echo "1. 환경변수 확인"
                    echo "CI_ENVIRONMENT_URL=$CI_ENVIRONMENT_URL"

                    echo "================================="
                    echo "2. DNS 확인"
                    getent hosts verdant-macaron-8d142a.netlify.app || true

                    echo "================================="
                    echo "3. HTTP 응답 확인"
                    curl -L -v "$CI_ENVIRONMENT_URL" -o /tmp/netlify.html

                    echo "================================="
                    echo "4. 실제 받은 HTML title 확인"
                    grep -i "<title" /tmp/netlify.html || true

                    echo "================================="
                    echo "5. HTML 앞부분"
                    head -30 /tmp/netlify.html

                    echo "================================="
                    echo "6. Playwright 실행"
                    npx playwright test --reporter=html
                '''
            }
        }
    }
}