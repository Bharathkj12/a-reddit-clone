pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME = "reddit-clone-pipeline"
        RELEASE = "1.0.0"
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/Bharathkj12/a-reddit-clone.git'
            }
        }
        stage("SonarQube Analysis") {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Reddit-Clone-CI \
                    -Dsonar.projectKey=Reddit-Clone-CI'''
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    waitForQualityGate abortPipeline: true, credentialsId: 'sonartoken'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('Prepare Cache Directory') {
            steps {
                script {
                    // Create the .cache directory for Jenkins user if it doesn't exist
                    def jenkinsHome = "/var/lib/jenkins"  // Adjust if your Jenkins home directory is different
                    def cacheDir = "${jenkinsHome}/.cache/trivy"

                    // Ensure the directory exists and set proper ownership and permissions
                    sh """
                        sudo mkdir -p ${cacheDir}
                        sudo chown -R jenkins:jenkins ${cacheDir}
                        sudo chmod -R 755 ${cacheDir}
                    """
                }
            }
        }
        stage('Trivy DB Update') {
            steps {
                script {
                    def cacheDir = "/var/lib/jenkins/.cache/trivy" // Use the same directory created for Jenkins user
                    // Set the cache directory environment variable for Trivy
                    withEnv(["TRIVY_CACHE_DIR=${cacheDir}"]) {
                        // Attempt to manually update the vulnerability DB
                        sh 'trivy --download-db-only || echo "Failed to download DB"'
                    }
                }
            }
        }
        stage('Trivy FS Scan') {
            steps {
                script {
                    def cacheDir = "/var/lib/jenkins/.cache/trivy" // Ensure the correct cache directory is used
                    withEnv(["TRIVY_CACHE_DIR=${cacheDir}"]) {
                        sh "trivy fs . > trivyfs.txt"
                    }
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
