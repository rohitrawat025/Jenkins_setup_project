pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out Terraform code from GitHub...'

                checkout scm
            }
        }

        stage('AWS Role Test') {
            steps {
                echo 'Checking AWS identity through Jenkins IAM role...'

                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkins_ru_aws'
                ]]) {
                    sh '''
                        set -e

                        echo "AWS identity:"
                        aws sts get-caller-identity
                    '''
                }
            }
        }

        stage('Terraform Version') {
            steps {
                sh '''
                    set -e

                    echo "Terraform version:"
                    terraform version
                '''
            }
        }

        stage('Terraform Format Check') {
            steps {
                sh '''
                    set -e

                    terraform fmt -check -recursive
                '''
            }
        }

        stage('Terraform Init') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkins_ru_aws'
                ]]) {
                    sh '''
                        set -e

                        terraform init
                    '''
                }
            }
        }

        stage('Terraform Validate') {
            steps {
                sh '''
                    set -e

                    terraform validate
                '''
            }
        }

        stage('Terraform Plan') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkins_ru_aws'
                ]]) {
                    sh '''
                        set -e

                        terraform plan -out=tfplan
                    '''
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input(
                    message: 'Terraform plan is ready. Do you want to create/update AWS infrastructure?',
                    ok: 'Apply Terraform'
                )
            }
        }

        stage('Terraform Apply') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkins_ru_aws'
                ]]) {
                    sh '''
                        set -e

                        terraform apply -auto-approve tfplan
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Terraform infrastructure deployment completed successfully.'
        }

        failure {
            echo 'Pipeline failed. Check the failed stage and console output.'
        }

        always {
            echo 'Jenkins pipeline execution completed.'
        }
    }
}
