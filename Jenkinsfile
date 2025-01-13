pipeline {
    agent any

    stages {
        stage('initial cleanup') {
            steps {
                dir('${WORKSPACE}') {
                    deleteDir()
                }
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/Simontagbor/php-todo.git'
            }
        }

        stage('prepare dependencies') {
            steps {
                sh 'mv .env.sample .env'
                sh 'composer update'	
                sh 'composer install'
                sh 'php artisan migrate'
                sh 'php artisan db:seed'
                sh 'php atisan key:generate'

            }
        }

        stage('Execute Unit Tests') {
            steps {
                sh './vendor/bin/phpunit'
            }
        }

        stage('Code Analysis') {
            steps {
                sh 'phploc app/ --log-csv build/logs/phploc.csv'
            }
        }

        stage('Plt Code Coverage Report') {
            steps {
                echo 'hello'

            }
        }

        stage('Package Artifact') {
            steps {
                sh 'zip -qr php-todo.zip ${WORKSPACE}/*'

            }
        }

        stage('Upload Artifact to Artifactory') {
            steps {
                script {
                    def server = Artifactory.server 'artifactory-server'
                    def uploadSpec = """{
                        "files": [
                            {
                                "pattern": "php-todo.zip",
                                "target": "jfrog-cli-remote-generic-local",
                                "props": "type=zip;status=ready"
                            }
                        ]
                    }"""
                    server.upload spec: uploadSpec
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                build job: 'ansible-dev/main', parameters: [[$class:'StringParameterValue', name: 'env', value:'dev']], propagate: false, wait: true
            }
        }
    }
}