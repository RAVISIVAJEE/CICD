
// Jenkinsfile for Vite CI/CD Pipeline

pipeline {
    // Agent specifies where the pipeline will run.
    // 'any' means Jenkins will run the pipeline on any available agent.
    // You can specify a label if you have specific agents (e.g., agent { label 'my-node-agent' }).
    agent any

    // Tools section to define NodeJS installation to be used
    tools {
        // This name must match the NodeJS installation name configured in Jenkins (Manage Jenkins -> Tools)
        nodejs 'NodeJS 24.4.1' // Changed from 'NodeJS 18' to 'NodeJS 20' to match Vite requirements
    }

    environment {
        // Define environment variables for SonarQube and S3
        // SonarQube project key (must match what you set in SonarQube)
        SONAR_PROJECT_KEY = 'My-Vite-Application'
        // SonarQube token ID from Jenkins Credentials
        SONAR_AUTH_TOKEN_ID = 'sonarqube-token'
        // AWS S3 bucket name
        S3_BUCKET_NAME = 'my-vite-app-cicd-demo-2025' // Replace with your S3 bucket name
        // AWS region for S3
        AWS_REGION = 'ap-east-1' // Replace with your AWS region
     
    }

    stages {
        // Stage 1: Source Code Checkout
        stage('Checkout Source Code') {
            steps {
                script {
                    echo 'Checking out source code from Git...'
                    // Checkout the source code from the configured Git repository
                    // The 'branch' parameter should be the actual branch name, not a refspec pattern.
                    git branch: 'main', credentialsId: '', url: 'https://github.com/RAVISIVAJEE/CICD.git' // Ensure this URL is correct for your repo
                }
            }
        }

        // Stage 2: Install NodeJS Dependencies
        stage('Install Dependencies') {
            steps {
                script {
                    echo 'Installing NodeJS dependencies...'
                    // Run npm install to get all project dependencies
                    sh 'npm install'
                }
            }
        }

        // Stage 3: Build Vite Application
        stage('Build Application') {
            steps {
                script {
                    echo 'Building Vite application...'
                    // Run npm build to create the production-ready build in the 'dist' folder
                    sh 'npm run build'
                }
            }
        }

        // Stage 4: Run Tests (Placeholder)
        // Vite does not come with a test runner by default.
        // You would integrate a framework like Vitest, Jest, or React Testing Library here.
        stage('Run Tests') {
            steps {
                script {
                    echo 'Running application tests (placeholder)...'
                    // Example with Vitest (you need to add Vitest to your project first: npm install -D vitest)
                    // sh 'npm test'
                    // For now, we'll just echo a message.
                    sh 'echo "No tests configured for this demo. Add Vitest or Jest to run tests here."'
                }
            }
        }

        // Stage 5: SonarQube Analysis
        stage('SonarQube Analysis') {
            steps {
                script {
                    echo 'Running SonarQube analysis...'
                    // Use the SonarQube Scanner plugin
                    // The 'sonar' step is provided by the SonarQube Scanner plugin
                    // The 'withSonarQubeEnv' step uses the globally configured SonarQube server.
                    // The 'sonarQubeUrl' parameter is not needed here as it's configured in Manage Jenkins -> System.
                    withSonarQubeEnv('SonarQube') {
                        sh """
                            npm install -g sonarqube-scanner # Install scanner if not already available
                            sonar-scanner \
                                -Dsonar.projectKey=${env.SONAR_PROJECT_KEY} \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=http://98.81.209.53:9000/ \
                                -Dsonar.token=${env.SONAR_AUTH_TOKEN_ID} \
                                -Dsonar.javascript.linter.eslint.reportPaths=eslint-report.json \
                                -Dsonar.javascript.linter.eslint.json.path=eslint-report.json
                        """
                        // Note: For actual ESLint integration, you'd need to configure ESLint in your Vite project
                        // and generate an eslint-report.json file before this step.
                        // For simplicity in this demo, it might just run basic analysis.
                    }
                }
            }
        }

        // Stage 6: Archive Artifacts
        stage('Archive Artifacts') {
            steps {
                script {
                    echo 'Archiving build artifacts...'
                    // Archive the 'dist' folder which contains the built application
                    archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
                }
            }
        }

        // Stage 7: Deploy to AWS S3
        stage('Deploy to S3') {
            steps {
                script {
                    echo "Deploying application to S3 bucket: ${env.S3_BUCKET_NAME}..."
                    // Use the AWS CLI for deployment. Ensure AWS CLI is installed on Jenkins agent.
                    // If using IAM Role on EC2, AWS CLI will automatically pick up credentials.
                    // If using IAM User, you'd configure credentials via environment variables or the AWS CLI config.
                    // For simplicity, we assume IAM Role or credentials configured globally.

                    // Install AWS CLI if not present (run this on your Jenkins EC2 instance manually or add to a pre-build step)
                    // sudo apt install awscli -y

                    // Sync the 'dist' folder to the S3 bucket
                    // --delete: Deletes files in S3 that are no longer in the source directory. Use with caution.
                    // --acl public-read: Makes uploaded objects publicly readable.
                    sh """
                        aws s3 sync dist/ s3://${env.S3_BUCKET_NAME}/ --delete --acl public-read --region ${env.AWS_REGION}
                    """
                    echo "Deployment to S3 complete. Access your application at: http://${env.S3_BUCKET_NAME}.s3-website.${env.AWS_REGION}.amazonaws.com"
                }
            }
        }
    }

    // Post-build actions (e.g., notifications, cleanup)
    post {
        always {
            echo 'Pipeline finished.'
            // Clean up workspace (optional, but good practice)
            // cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
