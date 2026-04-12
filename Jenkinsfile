pipeline {
    agent any
   
    environment {
        AWS_REGION = 'sa-east-1' 
    }
    stages {
        stage('Set AWS Credentials') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkinsuser' 
                ]]) {
                    sh '''
                    echo "AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID"
                    aws sts get-caller-identity
                    '''
                }
            }
        }
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/shinzo2x0/jenkins' 
            }
        }

    
        stage('Initialize Terraform') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkinsuser'
                ]]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform init
                    '''
                }
            }
        }

        stage('Plan Terraform') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'jenkinsuser'
                ]]) {
                    sh '''
                    export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
                    export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
                    terraform plan -out=tfplan
                    '''
                }
            }
        }
		stage('Apply Terraform') {
			steps {
				withCredentials([[
					$class: 'AmazonWebServicesCredentialsBinding',
					credentialsId: 'jenkinsuser'
				]]) {
					sh '''
					export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
					export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
					terraform apply -auto-approve tfplan
					'''
				}
			}
		}
		stage('Upload Images to S3') {
			steps {
				withCredentials([[
					$class: 'AmazonWebServicesCredentialsBinding',
					credentialsId: 'jenkinsuser'
				]]) {
					sh '''
					export AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
					export AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
					export AWS_DEFAULT_REGION=$AWS_REGION

					aws s3 sync pics/ s3://jenkins-bucket-20260330023333843200000001/pics/
					'''
				}
			}
		}
    }
    post {
        success {
            echo 'Terraform deployment completed successfully!'
        }
        failure {
            echo 'Terraform deployment failed!'
        }
    }
}