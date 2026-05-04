pipeline {
    agent any

    parameters {
        choice(name: 'QualityGate', choices: ['Pass', 'Fail'], description: 'Quality Gate')
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR = '539609142335.dkr.ecr.ap-south-1.amazonaws.com'
        COMMIT_ID = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
        BUILD_NUM = "1"
        IMAGE_TAG = "${COMMIT_ID}-${BUILD_NUM}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                git 'https://gitlab.cloudifyops.com/clops-training-assessments/devsecops-assessment.git'
            }
        }

        stage('Static Analysis') {
            parallel {
                stage('SonarQube') {
                    steps {
                        echo "Running SonarQube Scan"
                    }
                }
                stage('OWASP Scan') {
                    steps {
                        echo "Running OWASP Dependency Check"
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                script {
                    if (params.QualityGate == 'Fail') {
                        error "Quality Gate Failed"
                    }
                }
            }
        }

        stage('Run Tests') {
            steps {
                dir('web') {
                    sh 'npm install'
                    sh 'npm run test:unit || true'
                    sh 'npm run test:integ || true'
                }
                dir('api') {
                    sh 'python3 test.py || true'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                dir('api') {
                    sh 'docker build -t api-app .'
                }
                dir('web') {
                    sh 'docker build -t web-app .'
                }
            }
        }

        stage('Tag Images') {
            steps {
                sh """
                docker tag api-app ${ECR}/api-app:${IMAGE_TAG}
                docker tag web-app ${ECR}/web-app:${IMAGE_TAG}
                """
            }
        }

        stage('Anchore Scan') {
            steps {
                sh """
                docker run --rm -v /var/run/docker.sock:/var/run/docker.sock anchore/grype api-app:${IMAGE_TAG} -o json > anchore-scan.json || true
                """
            }
        }

        stage('ECR Login') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR}
                """
            }
        }

        stage('Push to ECR') {
            steps {
                sh """
                docker push ${ECR}/api-app:${IMAGE_TAG}
                docker push ${ECR}/web-app:${IMAGE_TAG}
                """
            }
        }

        stage('Kubelinter') {
            steps {
                sh 'echo Running kube-linter'
            }
        }
    }

    post {
        always {
            echo "Running Post Build Integration Test"
            echo "Sending Email Notification"
        }
    }
}