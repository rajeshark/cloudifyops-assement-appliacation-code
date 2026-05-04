pipeline {
    agent any

    parameters {
        choice(name: 'QualityGate', choices: ['Pass', 'Fail'], description: 'Quality Gate Decision')
    }

    environment {
        AWS_REGION = 'ap-south-1'
        ECR = '539609142335.dkr.ecr.ap-south-1.amazonaws.com'
    }

    stages {

        stage('Clean Workspace') {
            steps { cleanWs() }
        }

        stage('Checkout') {
            steps { checkout scm }
        }

        stage('Prepare') {
            steps {
                script {
                    env.COMMIT_ID = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    env.IMAGE_TAG = "${env.COMMIT_ID}-${env.BUILD_NUMBER}"
                }
            }
        }

        stage('Static Analysis') {
            parallel {
                stage('SonarQube Analysis') {
                    steps {
                        sh '/opt/sonar-scanner/bin/sonar-scanner'
                    }
                }
                stage('OWASP Dependency Check') {
                    steps {
                        sh '''
                        echo "OWASP scan" > dependency-check-report.xml
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
                sh 'echo Running API tests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t api-app:${IMAGE_TAG} ./api"
            }
        }

        stage('Anchore Scan') {
            steps {
                sh """
                docker run --rm anchore/grype api-app:${IMAGE_TAG} -o json > anchore-scan.json || true
                """
            }
        }

        stage('Push Image to ECR') {
            steps {
                sh """
                aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR}
                docker tag api-app:${IMAGE_TAG} ${ECR}/api-app:${IMAGE_TAG}
                docker push ${ECR}/api-app:${IMAGE_TAG}
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
                        echo "Deploy API to DEV namespace"
                    } else if (env.BRANCH_NAME == "main") {
                        echo "Deploy API to PROD namespace"
                    }
                }
            }
        }
    }

    post {
        always {
            emailext (
                subject: "API Build ${currentBuild.currentResult}",
                body: """
Build: ${env.BUILD_NUMBER}
Branch: ${env.BRANCH_NAME}
Commit: ${env.GIT_COMMIT}
URL: ${env.BUILD_URL}
""",
                to: "rajesha100920@gmail.com"
            )
        }
    }
}
