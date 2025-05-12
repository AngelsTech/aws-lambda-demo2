pipeline {
    agent any

    parameters {
        string(name: 'GIT_BRANCH', defaultValue: 'main', description: 'Branch to build')
    }

    environment {
        LAMBDA_REPO_URL = 'https://github.com/AngelsTech/aws-lambda-demo.git'
        LAMBDA_FUNCTION_NAME = 'my-lambda-function'
    }

    stages {
        stage('Clone Lambda Repo') {
            steps {
                dir('lambda-function') {
                    git branch: "${params.GIT_BRANCH}", url: "${env.LAMBDA_REPO_URL}"
                }
            }
        }

        stage('Install Dependencies & Package') {
            steps {
                dir('lambda-function') {
                    sh '''
                    mkdir -p build
                    pip install -r requirements.txt -t build/
                    cp *.py build/
                    cd build
                    zip -r ../function.zip .
                    '''
                }
            }
        }

        stage('Deploy to AWS Lambda') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-credentials-id'
                ]]) {
                    dir('lambda-function') {
                        sh '''
                        aws lambda update-function-code \
                            --function-name ${LAMBDA_FUNCTION_NAME} \
                            --zip-file fileb://function.zip
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Lambda deployment successful!'
        }
        failure {
            echo '❌ Lambda deployment failed.'
        }
    }
}
