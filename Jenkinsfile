pipeline {
    agent any
    environment {
        version = 'V1'
    }
    stages {
        stage('Download Code from GHub') {
            steps {
                sh 'git clone https://github.com/cobidennis/hrapp.git'
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
        stage ('Ansible: Deploy Apps and Monitoring System') {
            steps {
                sh '''
                    git clone https://github.com/cobidennis/hrapp-nodes-config.git
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