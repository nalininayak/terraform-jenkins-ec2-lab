pipeline {
    agent any

    stages {

        stage('Test SSH to EC2') {
            steps {

                withCredentials([
                    file(
                        credentialsId: 'ec2-private-key',
                        variable: 'SSH_KEY'
                    )
                ]) {

                    bat '''
                        echo Testing SSH connectivity...

                        icacls "%SSH_KEY%" /inheritance:r
                        icacls "%SSH_KEY%" /grant:r "SYSTEM:R"

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@13.201.21.209 "hostname"
                    '''
                }
            }
        }

        stage('Install Nginx on EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-private-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    bat '''
                        ssh -o BatchMode=yes ^
                            -o StrictHostKeyChecking=no ^
                            -o ConnectTimeout=10 ^
                            -i "%SSH_KEY%" ^
                            ubuntu@3.108.63.81 ^
                            "sudo apt-get update &&
                             sudo apt-get install -y nginx curl jq &&
                             sudo systemctl enable --now nginx &&
                             sudo systemctl is-active nginx"
                    '''
                }
            }
        }

        stage('Validate Nginx') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-private-key',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    bat '''
                        ssh -o BatchMode=yes ^
                            -o StrictHostKeyChecking=no ^
                            -o ConnectTimeout=10 ^
                            -i "%SSH_KEY%" ^
                            ubuntu@13.201.21.209 ^
                            "systemctl is-active nginx &&
                             curl -fsS http://127.0.0.1/ > /dev/null &&
                             echo NGINX_HTTP_HEALTHCHECK_OK"
                    '''
                }
            }
        }
    }
}
