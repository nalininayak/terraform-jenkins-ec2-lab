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
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "%USERDOMAIN%\%USERNAME%:R"
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "SYSTEM:R"
                        if errorlevel 1 exit /b 1

                        ssh -o StrictHostKeyChecking=no -i "%SSH_KEY%" ubuntu@3.108.63.81 "hostname"

                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Install Nginx on EC2') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'ec2-private-key',
                        variable: 'SSH_KEY'
                    )
                ]) {
                    bat '''
                        echo Securing temporary SSH key...

                        icacls "%SSH_KEY%" /inheritance:r
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "%USERDOMAIN%\%USERNAME%:R"
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "*S-1-5-18:R"
                        if errorlevel 1 exit /b 1

                        echo Checking key ACL...
                        icacls "%SSH_KEY%"
                        if errorlevel 1 exit /b 1

                        echo Installing Nginx on EC2...

                        ssh -o BatchMode=yes -o StrictHostKeyChecking=no -o ConnectTimeout=10 -i "%SSH_KEY%" ubuntu@3.108.63.81 "sudo apt-get update && sudo apt-get install -y nginx curl jq && sudo systemctl enable --now nginx && sudo systemctl is-active nginx"

                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }

        stage('Validate Nginx') {
            steps {
                withCredentials([
                    file(
                        credentialsId: 'ec2-private-key',
                        variable: 'SSH_KEY'
                    )
                ]) {
                    bat '''
                        icacls "%SSH_KEY%" /inheritance:r
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "%USERDOMAIN%\%USERNAME%:R"
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "*S-1-5-18:R"
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%"
                        if errorlevel 1 exit /b 1

                        ssh -o BatchMode=yes -o StrictHostKeyChecking=no -o ConnectTimeout=10 -i "%SSH_KEY%" ubuntu@3.108.63.81 "systemctl is-active nginx && curl -fsS http://127.0.0.1/ > /dev/null && echo NGINX_HTTP_HEALTHCHECK_OK"

                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }
    }
}
