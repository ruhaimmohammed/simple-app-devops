pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        booleanParam(name: 'PUSH_DOCKER', defaultValue: true, description: 'Build image?')
        string(name: 'DOCKER_IMAGE', defaultValue: 'simple-python-app', description: 'Image name')
    }

    environment {
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                cleanWs() 
                checkout([$class: 'GitSCM',
                          branches: [[name: "refs/heads/${params.BRANCH}"]],
                          userRemoteConfigs: [[url: "https://github.com/ruhaimmohammed/simple-app-devops.git"]]])
            }
        }

        stage('Build') {
            steps {
                script {
                    // Optimized for your Python files found in 'ls -al'
                    if (fileExists('requirement.txt')) {
                        echo 'Detected Python project'
                        // If you have python installed on the server, you can test the install
                        // sh 'pip install -r requirement.txt'
                    } else if (fileExists('app.py')) {
                        echo 'Detected Flask/Python app'
                    } else {
                        echo 'No recognized build file found.'
                    }
                }
            }
        }

        stage('Docker Build') {
            when { allOf { expression { fileExists('Dockerfile') }; expression { return params.PUSH_DOCKER } } }
            steps {
                script {
                    def tag = "${params.DOCKER_IMAGE}:${env.BUILD_TAG}"
                    // Building the image locally on the EC2
                    sh "docker build -t ${tag} ."
                    sh "docker tag ${tag} ${params.DOCKER_IMAGE}:latest"
                    echo "Docker image built: ${tag}"
                }
            }
        }

        stage('Deploy Local') {
            when { expression { return params.PUSH_DOCKER } }
            steps {
                script {
                    echo "Deploying with Docker Compose..."
                    // Stop and remove old containers, then start new ones in the background
                    sh "docker-compose down || true"
                    sh "docker-compose up -d"
                }
            }
        }
    }

    post {
        success {
            echo "Build ${env.BUILD_NUMBER} succeeded. App is running on port 5000."
        }
        failure {
            echo "Build ${env.BUILD_NUMBER} failed. Check console logs."
        }
        always {
            // We keep the image on the server but clean the code files
            cleanWs()
        }
    }
}