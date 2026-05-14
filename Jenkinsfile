pipeline {
    agent any

     environment {
        REGISTRY = 'localhost:5000'
        APP_NAME = 'market-data-service'
        IMAGE_TAG = "${BUILD_NUMBER}"
        FULL_IMAGE = "${REGISTRY}/${APP_NAME}:v${IMAGE_TAG}"
    }


    options {
        timestamps()
    }

    stages {

        stage('Checkout Application Repo') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Show Workspace') {
            steps {
                sh '''
                    pwd
                    ls -la
                    ls -la market-data-service
                '''
            }
        }

        stage('Configure Release') {
            steps {
                dir('market-data-service') {
                    sh '''
                        cmake --preset release
                    '''
                }
            }
        }

        stage('Build Release') {
            steps {
                dir('market-data-service') {
                    sh '''
                        cmake --build --preset release
                    '''
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                dir('market-data-service') {
                    sh '''
                        ctest --preset release
                    '''
                }
            }
        }

        stage('Static Analysis') {
            steps {
                dir('market-data-service') {
                    sh '''
                        cppcheck src include \
                          --enable=warning,performance,portability \
                          || true
                    '''
                }
            }
        }
		
	stage('Sanitizer Build') {
    when {
        branch 'main'
    }

    steps {
        dir('market-data-service') {
            sh '''
                cmake --preset asan
                cmake --build --preset asan
                ctest --preset asan
            '''
        }
    }
}
        
stage('Approval Gate') {
    when {
        branch 'main'
    }

    steps {
        input 'CI passed. Approve Docker build?'
    }
}
		
		
		
stage('Docker Build') {
    when {
        branch 'main'
    }

    steps {
        dir('market-data-service') {
            sh '''
                docker build -t $FULL_IMAGE .
            '''
        }
    }
}

stage('Security Scan - Trivy') {
    when {
        branch 'main'
    }

    steps {
        sh '''
            trivy image \
              --severity HIGH,CRITICAL \
              --exit-code 0 \
              --no-progress \
              $FULL_IMAGE
        '''
    }
}

stage('Push Image to Private Registry') {
    when {
        branch 'main'
    }

    steps {
        sh '''
            docker push $FULL_IMAGE
        '''
    }
}
    }

    post {
        success {
            echo 'completed successfully.'
        }

        failure {
            echo 'Pipeline failed.'
        }
    }
}
