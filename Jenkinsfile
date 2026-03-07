pipeline {
    agent {
        label 'AGENT-1'
    }

    environment { 
        appVersion = ''
        REGION = 'us-east-1'
        ACC_ID = '246816820124'
        PROJECT = 'roboshop'
        COMPONENT = 'catalogue'
    }

    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }

    stages {

        stage('Read Package.json') {
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "package version: ${appVersion}"
                }
            }
        }

        stage('install dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Docker build') {
            steps {
                script {
                    withAWS(credentials: 'aws-creds', region: 'us-east-1') {
                        sh """
                        aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com
                        docker build -t ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion} .
                        docker push ${ACC_ID}.dkr.ecr.${REGION}.amazonaws.com/${PROJECT}/${COMPONENT}:${appVersion}
                        """
                    }
                }
            }
        }

        stage('Trigger Deploy') {
            when {
                expression { params.deploy }
            }
            steps {
                build job: 'catalogue-cd',
                parameters: [
                    string(name: 'appVersion', value: "${appVersion}"),
                    string(name: 'deploy_to', value: 'dev')
                ],
                propagate: false,
                wait: false
            }
        }
    }

    post {
        always {
            echo 'I will always say Hello again!'
            deleteDir()
        }

        success {
            echo 'Hello Success'
        }

        failure {
            echo 'Hello Failure'
        }
    }
}
