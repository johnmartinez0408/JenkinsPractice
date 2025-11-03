pipeline {
    agent any

    environment {
        APP_NAME = 'static-site'
        IMAGE_TAG = "${env.BUILD_NUMBER}"          // e.g., 15
        DOCKER_IMAGE = "local/${env.APP_NAME}:${env.IMAGE_TAG}"
        HOST_PORT = '8081'                          // change if you want a different port
        CONTAINER_NAME = "${env.APP_NAME}"
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
          docker build -t local/%APP_NAME%:%IMAGE_TAG% .
        '''
            }
        }

        stage('Stop & Remove Old Container (if exists)') {
            steps {
                bat '''
          for /f "tokens=*" %%i in ('docker ps -aq -f "name=%APP_NAME%"') do (
            docker rm -f %%i || echo "No existing container found"
          )
        '''
            }
        }

        stage('Run New Container') {
            steps {
                bat '''
          docker run -d --name %APP_NAME% -p %HOST_PORT%:80 local/%APP_NAME%:%IMAGE_TAG%
        '''
            }
        }

        stage('Health Check') {
            steps {
                script {
                    // Try curl a few times to allow container to start
                    def tries = 10
                    def ok = false
                    for (int i = 0; i < tries; i++) {
                        def code = bat(
              script: 'curl -s -o NUL -w "%{http_code}" http://localhost:%HOST_PORT%',
              returnStdout: true
            ).trim()

                        if (code == '200') {
                            ok = true
                            break
                        }
                        sleep 2
                    }
                    if (!ok) {
                        error "Health check failed: site not responding on http://localhost:${HOST_PORT}"
                    }
                }
            }
        }
    }

    post {
        always {
            bat '''
      echo === Docker Images ===
      docker images || exit /b 0

      echo === Running Containers ===
      docker ps -a || exit /b 0
    '''
        }
    }
}
