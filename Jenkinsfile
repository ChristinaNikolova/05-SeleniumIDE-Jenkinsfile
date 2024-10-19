pipeline {
    agent any
    environment {
        CHROME_VERSION = '130.0.6723.5800'
        CHROMEDRIVER_VERSION = '130.0.6723.5800'
        CHROME_INSTALL_PATH = 'C:\\Program Files\\Google\\Chrome\\Application'
        CHROMEDRIVER_PATH = '"C:\\Program Files\\Google\\Chrome\\Application\\chromedriver.exe"'
    }
    stages {
        stage('Checkout the repo') {
            steps {
                git branch: 'master', url: 'https://github.com/ChristinaNikolova/05-SeleniumIDE-Jenkinsfile' // Fixed 'brach' to 'branch'
            }
        }
        stage('Install Chocolatey') {
            steps {
                script {
                    def chocoInstalled = bat(script: 'choco --version', returnStatus: true) == 0
                    if (!chocoInstalled) {
                        echo 'Chocolatey is not installed. Installing Chocolatey...'
                        bat '''
                        powershell -Command "Invoke-WebRequest -Uri https://chocolatey.org/install.ps1 -OutFile install.ps1"
                        powershell -Command "Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((Get-Content .\\install.ps1 -Raw))"
                        '''
                    } else {
                        echo 'Chocolatey is already installed.'
                    }
                }
            }
        }
        stage('Setup .NET Core') {
            steps {
                script {
                    def sdkInstalled = bat(script: 'choco list --local-only dotnet-sdk', returnStatus: true) == 0
                    if (sdkInstalled) {
                        echo '.NET SDK is already installed.'
                    } else {
                        echo 'Installing .NET SDK...'
                        bat 'choco install dotnet-sdk -y --version=6.0.100'
                    }
                }
            }
        }
        stage('Uninstall current Google Chrome') {
            steps {
               script {
                    def chromeInstalled = bat(script: 'choco list --local-only googlechrome', returnStatus: true) == 0
                    if (chromeInstalled) {
                        echo 'Uninstalling Google Chrome...'
                        bat 'choco uninstall googlechrome -y'
                    } else {
                        echo 'Google Chrome is not installed. Skipping uninstallation.'
                    }
                }
            }
        }
        stage('Install Google Chrome') {
            steps {
                bat '''
                echo Installing Google Chrome version %CHROME_VERSION%
                choco install googlechrome --version=%CHROME_VERSION% -y --allow-downgrade --ignore-checksums
                '''
            }
        }
        stage('Download and Install ChromeDriver') {
            steps {
                bat '''
                echo Downloading ChromeDriver version %CHROMEDRIVER_VERSION%
                powershell -command "Invoke-WebRequest -Uri https://chromedriver.storage.googleapis.com/%CHROMEDRIVER_VERSION%/chromedriver_win32.zip -OutFile chromedriver.zip -UseBasicParsing"
                powershell -command "Expand-Archive -Path chromedriver.zip -DestinationPath ."
                powershell -command "Move-Item -Path .\\chromedriver.exe -Destination '%CHROME_INSTALL_PATH%\\chromedriver.exe' -Force"
                '''
            }
        }
        stage('Restore dependencies') {
            steps {
                bat 'dotnet restore SeleniumIde.sln'
            }
        }
        stage('Build the application') {
            steps {
                bat 'dotnet build SeleniumIde.sln --configuration Release'
            }
        }
        stage('Run the tests') {
            steps {
                bat 'dotnet test SeleniumIde.sln --logger "trx;LogFileName=TestResults.trx"'
            }
        }
        // post {
        //     always {
        //         archiveArtifacts artifacts: '**/TestResults/*.trx', allowEmptyArchive: true
        //         junit '**/TestResults/*.trx'
        //     }
        // }
    }
}