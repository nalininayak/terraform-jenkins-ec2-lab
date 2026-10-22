pipeline {
    agent any

    stages {
        stage('Test SSH to EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-private-key-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    bat '''
                        @echo off
                        setlocal

                        echo Testing SSH connectivity...
                        echo Applying secure ACL to temporary key...

                        REM Disable inheritance and remove inherited ACEs
                        icacls "%SSH_KEY%" /inheritance:r
                        if errorlevel 1 exit /b 1

                        REM Remove access granted to broad user groups
                        icacls "%SSH_KEY%" /remove:g *S-1-5-32-545 *S-1-5-11 *S-1-1-0
                        if errorlevel 1 exit /b 1

                        REM Grant read access to Local System
                        icacls "%SSH_KEY%" /grant:r "*S-1-5-18:(R)"
                        if errorlevel 1 exit /b 1

                        echo Verifying key ACL...
                        icacls "%SSH_KEY%"
                        if errorlevel 1 exit /b 1

                        echo Testing SSH login...

                        ssh -o BatchMode=yes ^
                            -o ConnectTimeout=10 ^
                            -o StrictHostKeyChecking=no ^
                            -i "%SSH_KEY%" ^
                            %SSH_USER%@3.108.63.81 "hostname"

                        if errorlevel 1 exit /b 1

                        echo SSH test completed successfully.
                    '''
                }
            }
        }

        stage('Install Nginx on EC2') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-private-key-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    bat '''
                        @echo off
                        setlocal

                        echo Installing Nginx on EC2...

                        ssh -o BatchMode=yes ^
                            -o StrictHostKeyChecking=no ^
                            -o ConnectTimeout=10 ^
                            -i "%SSH_KEY%" ^
                            %SSH_USER%@3.108.63.81 ^
                            "sudo apt-get update && sudo apt-get install -y nginx curl"

                        if errorlevel 1 exit /b 1

                        echo Nginx installation command completed.
                    '''
                }
            }
        }

        stage('Validate Nginx') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ec2-private-key-ssh',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    bat '''
                        @echo off
                        setlocal

                        icacls "%SSH_KEY%" /inheritance:r
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "%USERDOMAIN%\\%USERNAME%:R"
                        if errorlevel 1 exit /b 1

                        icacls "%SSH_KEY%" /grant:r "*S-1-5-18:(R)"
                        if errorlevel 1 exit /b 1

                        ssh -o BatchMode=yes ^
                            -o StrictHostKeyChecking=no ^
                            -o ConnectTimeout=10 ^
                            -i "%SSH_KEY%" ^
                            %SSH_USER%@3.108.63.81 ^
                            "systemctl is-active nginx && curl -fsS http://127.0.0.1/ > /dev/null && echo NGINX_HTTP_HEALTHCHECK_OK"

                        if errorlevel 1 exit /b 1
                    '''
                }
            }
        }
    }
}
