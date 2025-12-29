pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        // Changed default to true so you can see it work
        booleanParam(name: 'PUSH_DOCKER', defaultValue: true, description: 'Build and push Docker image?')
        string(name: 'DOCKER_IMAGE', defaultValue: 'myorg/simple-app', description: 'Image name (repo/name)')
    }

    environment {
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                // CLEAN the workspace BEFORE downloading new code
                cleanWs() 
                checkout([$class: 'GitSCM',
                          branches: [[name: "refs/heads/${params.BRANCH}"]],
                          userRemoteConfigs: [[url: "https://github.com/ruhaimmohammed/simple-app-devops.git"]]])
            }
        }

        stage('Prepare') {
            steps {
                // Removed cleanWs() from here!
                echo "Building branch: ${params.BRANCH}, build #: ${env.BUILD_NUMBER}"
                sh 'ls -al' // Useful for debugging: see if your files are actually there
            }
        }

        stage('Build') {
            steps {
                script {
                    // Tip: Ensure these files (pom.xml, etc.) are in the ROOT of your GitHub repo
                    if (fileExists('pom.xml')) {
                        echo 'Detected Maven project'
                        sh 'mvn -B -DskipTests package'
                    } else if (fileExists('package.json')) {
                        echo 'Detected Node.js project'
                        sh 'npm install && npm run build --if-present'
                    } else if (fileExists('index.html')) {
                        echo 'Detected simple Static HTML project'
                        // Static apps don't need a "build" command usually
                    } else {
                        echo 'No recognized build file found. Listing files to debug:'
                        sh 'ls -R'
                    }
                }
            }
        }

        stage('Docker Build & Push') {
            // This will now run if a Dockerfile exists and the parameter is true
            when { allOf { expression { fileExists('Dockerfile') }; expression { return params.PUSH_DOCKER } } }
            steps {
                script {
                    def tag = "${params.DOCKER_IMAGE}:${env.BUILD_TAG}"
                    sh "docker build -t ${tag} ."
                    echo "Docker image built: ${tag}"
                    // Note: 'docker push' will fail unless you've run 'docker login' or added credentials
                }
            }
        }
    }

    post {
        success {
            echo "Build ${env.BUILD_NUMBER} succeeded."
        }
        failure {
            echo "Build ${env.BUILD_NUMBER} failed."
        }
        always {
            // Keep this here to clean up AFTER the build is done
            cleanWs()
        }
    }
}