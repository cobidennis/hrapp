pipeline {
    agent any
    environment {
        version = 'V1'
    }
    stages {
        stage('Download Code from GHub') {
            steps {
                sh '''
                    mkdir workspace
                    cd workspace
                    git clone https://github.com/cobidennis/hrapp.git
                    git clone https://github.com/cobidennis/hrapp-nodes-config.git
                '''

            }
        }
        stage ('Build Image') {
            steps {
                sh '''
                    cd workspace/hrapp
                    docker build -t cobidennis/hrapp:$version .
                    docker tag cobidennis/hrapp:$version cobidennis/hrapp:release
                '''
            }
        }
        stage ('Push Image to DHub') {
            steps {
                withCredentials([
                    [
                        $class: 'UsernamePasswordMultiBinding',
                        credentialsId:'docker',
                        usernameVariable: 'USERNAME',
                        passwordVariable: 'PASSWORD'
                    ]
                ]) {
                    sh '''
                        docker login -u $USERNAME -p $PASSWORD
                        docker push cobidennis/hrapp:$version
                        docker push cobidennis/hrapp:release
                    '''
                } 
            }
        }
        stage('Download Env File from S3') {
            steps {
                script {
                    def s3Bucket = 'dobee-buckets'
                    def hrappEnv = 'devops-training/terraform/hr-app/hrapp.yml'
                    def keyFile = 'keys/DobeeP53.pem'
                    def envFilePath = 'workspace/hrapp-nodes-config/ansible/env_files'

                    sh "aws s3 cp s3://${s3Bucket}/${hrappEnv} ${envFilePath}"
                    sh "aws s3 cp s3://${s3Bucket}/${keyFile} workspace"

                    def hrappEnvExists = fileExists("${envFilePath}/hrapp.yml")
                    def keyFileExists = fileExists("workspace/DobeeP53.pem")

                    if (hrappEnvExists && keyFileExists) {
                        echo "HR App Env and Key File Exists. Proceeding to the next stage."
                    } else {
                        error "HR App Env/Key File Not Found. Aborting the pipeline."
                    }
                }
            }
        }
        stage ('Ansible: Deploy Apps and Monitoring System') {
            when {
                expression {
                    return hrappEnvExists && keyFileExists
                }
            }
            steps {
                sh '''
                    cd workspace/hrapp-nodes-config/ansible
                    ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook -e @./env_files/hrapp.yml -i inventory.ini  playbook-app.yml --key-file ../../DobeeP53.pem -u ec2-user
                '''
            }
        }
    }
    post {
        always {
            deleteDir()
        }
    }
}