// Jenkinsfile for Vite CI/CD Pipeline

pipeline {
    agent any

    tools {
        nodejs 'NodeJS 24.4.1' // Make sure this matches your Jenkins NodeJS installation name
    }

    environment {
        SONAR_PROJECT_KEY = 'My-Vite-Application' // Change if your SonarQube project key is different
        SONAR_AUTH_TOKEN_ID = 'sonarqube-token' // Make sure this matches your Jenkins credentials ID for SonarQube token
        S3_BUCKET_NAME = 'my-vite-app-cicd-demo-2025' // Replace with your actual S3 bucket name
        AWS_REGION = 'us-east-1' // Replace with your AWS region
        // Replace if using IAM User credentials, or remove if using IAM Role
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                script {
                    echo 'Checking out source code from Git...'
                    git branch: 'main', credentialsId: '', url: 'https://github.com/RAVISIVAJEE/CICD.git'
                    // Replace 'your-username/my-vite-app.git' with your actual repo URL
                    // If your repo is private, set credentialsId to your Jenkins Git credentials ID
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    echo 'Installing NodeJS dependencies...'
                    sh 'npm install'
                }
            }
        }

        stage('Build Application') {
            steps {
                script {
                    echo 'Building Vite application...'
                    sh 'npm run build'
                }
            }
        }

        stage('Run Tests') {
            steps {
                script {
                    echo 'Running application tests (placeholder)...'
                    sh 'echo "No tests configured for this demo. Add Vitest or Jest to run tests here."'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Running SonarQube analysis...'
                    // Use the SonarQube Scanner plugin
                    // The 'sonar' step is provided by the SonarQube Scanner plugin
                    // The 'withSonarQubeEnv' step uses the globally configured SonarQube server.
                    // The 'sonarQubeUrl' parameter is not needed here as it's configured in Manage Jenkins -> System.
                    withSonarQubeEnv(credentialsId: "${env.SONAR_AUTH_TOKEN_ID}") {
                        sh """
                            /usr/bin/npm install -g sonarqube-scanner # Install scanner if not already available
                            sonar-scanner \
                                -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://<your-sonarqube-ec2-public-ip>:9000 \
                                -Dsonar.login=${env.SONAR_AUTH_TOKEN_ID} \
                                -Dsonar.javascript.linter.eslint.reportPaths=eslint-report.json \
                                -Dsonar.javascript.linter.eslint.json.path=eslint-report.json
                        """
                        // Note: For actual ESLint integration, you'd need to configure ESLint in your Vite project
                        // and generate an eslint-report.json file before this step.
                      
                }
            }
        }

        stage('Archive Artifacts') {
            steps {
                script {
                    echo 'Archiving build artifacts...'
                    archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
                }
            }
        }

        stage('Deploy to S3') {
            steps {
                script {
                    echo "Deploying application to S3 bucket: ${env.S3_BUCKET_NAME}..."
                    sh """
                        aws s3 sync dist/ s3://${env.S3_BUCKET_NAME}/ --delete --acl public-read --region ${env.AWS_REGION}
                    """
                    echo "Deployment to S3 complete. Access your application at: http://${env.S3_BUCKET_NAME}.s3-website.${env.AWS_REGION}.amazonaws.com"
                    // Make sure your S3 bucket exists and is configured for static website hosting
                    // Ensure AWS CLI is installed on your Jenkins agent
                    // If using IAM User, configure credentials; if using IAM Role, make sure permissions are correct
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline finished.'
            // Optionally uncomment cleanWs() if you want to clean workspace
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
