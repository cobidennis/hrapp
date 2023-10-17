pipeline {
    agent any
    environment {
        version = 'V1'
    }
    stages {
        stage('Download Code from GHub') {
            steps {
                sh '''
                    git clone https://github.com/cobidennis/hrapp.git
                    git clone https://github.com/cobidennis/hrapp-nodes-config.git
                '''

            }
        }
        stage ('Build Image') {
            steps {
                sh '''
                    cd hrapp
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
                    def localFilePath = 'hrapp-nodes-config/ansible/env_files'

                    sh "aws s3 cp s3://${s3Bucket}/${hrappEnv} ${localFilePath}"

                    def hrappEnvExists = fileExists("${localFilePath}/hrapp.yml")

                    if (hrappEnvExists) {
                        echo "HR App Env File Exists. Proceeding to the next stage."
                    } else {
                        error "HR App Env File Not Found. Aborting the pipeline."
                    }
                }
            }
        }
        stage ('Ansible: Deploy Apps and Monitoring System') {
            when {
                expression {
                    return hrappEnvExists
                }
            }
            steps {
                sh '''
                    cd hrapp-nodes-config/ansible
                    ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook -e @/tmp/hrapp.yml -i /tmp/inventory.ini  playbook.yml --key-file /tmp/DobeeP53.pem -u ec2-user
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