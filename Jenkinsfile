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
    }
}
