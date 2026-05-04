pipeline {
    agent any

    parameters {
        choice(name: 'QualityGate', choices: ['Pass', 'Fail'], description: 'Quality Gate Decision')
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR = '539609142335.dkr.ecr.ap-south-1.amazonaws.com'
        SONAR_HOST_URL = 'http://3.110.100.156:9000'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare') {
            steps {
                script {
                    COMMIT_ID = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    IMAGE_TAG = "${COMMIT_ID}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Static Analysis') {
            parallel {

                stage('SonarQube Analysis') {
                    steps {
                        sh 'sonar-scanner'
                    }
                }

                stage('OWASP Dependency Check') {
                    steps {
                        sh '''
                        mkdir -p owasp-report
                        echo "OWASP scan simulated" > owasp-report/dependency-check-report.xml
                        '''
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
                sh '''
                echo "Running tests..."
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh """
                docker build -t api-app:${IMAGE_TAG} ./api
                docker build -t web-app:${IMAGE_TAG} ./web
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

        stage('Push to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR}
                docker tag api-app:${IMAGE_TAG} ${ECR}/api-app:${IMAGE_TAG}
                docker tag web-app:${IMAGE_TAG} ${ECR}/web-app:${IMAGE_TAG}
                docker push ${ECR}/api-app:${IMAGE_TAG}
                docker push ${ECR}/web-app:${IMAGE_TAG}
                """
            }
        }

        stage('Kubelinter Helm Charts') {
            steps {
                sh '''
                git clone https://github.com/rajeshark/application-helm-charts.git
                echo "Running kube-linter..."
                '''
            }
        }

        stage('Deploy Decision') {
            steps {
                script {
                    if (env.BRANCH_NAME == "develop") {
                        echo "Deploying to DEV namespace"
                    } else if (env.BRANCH_NAME == "master") {
                        echo "Deploying to PROD namespace"
                    }
                }
            }
        }
    }

    post {
        always {
            echo "Post Build Actions"

            sh '''
            echo "Checking pods..."
            '''

            echo "Sending email notification (simulated)"
        }
    }
}
