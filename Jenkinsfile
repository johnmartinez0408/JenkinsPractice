pipeline {
    agent any

    environment {
        APP_NAME = 'static-site'
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKER_IMAGE = "local/static-site:${BUILD_NUMBER}"
        HOST_PORT = '8081'
        CONTAINER_NAME = 'static-site'
    }

    options { timestamps() }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    echo Building Docker image...
                    docker build -t local/static-site:%BUILD_NUMBER% .
                    if %errorlevel% neq 0 exit /b %errorlevel%
                '''
            }
        }

        stage('Stop & Remove Old Container (if exists)') {
            steps {
                bat '''
                    echo Stopping and removing any existing container...
                    for /f "tokens=*" %%i in ('docker ps -aq -f "name=static-site"') do (
                        docker rm -f %%i
                    )
                    exit /b 0
                '''
            }
        }

        stage('Run New Container') {
            steps {
                bat '''
                    echo Running new container...
                    docker run -d --name static-site -p 8081:80 local/static-site:%BUILD_NUMBER%
                    if %errorlevel% neq 0 exit /b %errorlevel%
                '''
            }
        }

        stage('Health Check') {
            steps {
                script {
                    def ok = false
                    for (int i = 0; i < 10; i++) {
                        def output = bat(
                            script: 'powershell -Command "(Invoke-WebRequest -UseBasicParsing http://localhost:8081).StatusCode" 2>$null',
                            returnStdout: true
                        ).trim()

                        if (output == '200') {
                            ok = true
                            break
                        }
                        sleep 2
                    }
                    if (!ok) {
                        error "Health check failed: site not responding on http://localhost:8081"
                    }
                }
            }
        }
    }

    post {
        always {
            echo '=== Docker Images ==='
            bat(returnStatus: true, script: 'docker images')

            echo '=== Running Containers ==='
            bat(returnStatus: true, script: 'docker ps -a')

            // Explicitly mark build as success if no error() was triggered
            script {
                if (currentBuild.result == null || currentBuild.result == 'SUCCESS') {
                    currentBuild.result = 'SUCCESS'
                }
            }
        }
    }
}
