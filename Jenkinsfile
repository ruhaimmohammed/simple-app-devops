pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
        booleanParam(name: 'RUN_TESTS', defaultValue: true, description: 'Run tests?')
        booleanParam(name: 'PUSH_DOCKER', defaultValue: false, description: 'Build and push Docker image?')
        string(name: 'DOCKER_IMAGE', defaultValue: 'myorg/simple-app', description: 'Image name (repo/name)')
    }

    environment {
        BUILD_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                                    branches: [[name: "refs/heads/${params.BRANCH}"]],
                                    userRemoteConfigs: [[url: env.GIT_URL ?: scm.userRemoteConfigs[0].url]]])
            }
        }

        stage('Prepare') {
            steps {
                cleanWs()
                echo "Building branch: ${params.BRANCH}, build #: ${env.BUILD_NUMBER}"
            }
        }

        stage('Build') {
            steps {
                script {
                    if (fileExists('pom.xml')) {
                        echo 'Detected Maven project'
                        sh 'mvn -B -DskipTests package'
                    } else if (fileExists('build.gradle') || fileExists('build.gradle.kts')) {
                        echo 'Detected Gradle project'
                        sh './gradlew build -x test --no-daemon || gradle build -x test'
                    } else if (fileExists('package.json')) {
                        echo 'Detected Node.js project'
                        sh 'npm ci && npm run build --if-present'
                    } else {
                        echo 'No recognized build file found; skipping build step'
                    }
                }
            }
        }

        stage('Test') {
            when { expression { return params.RUN_TESTS } }
            steps {
                script {
                    if (fileExists('pom.xml')) {
                        sh 'mvn -B test'
                    } else if (fileExists('build.gradle') || fileExists('build.gradle.kts')) {
                        sh './gradlew test --no-daemon || gradle test'
                    } else if (fileExists('package.json')) {
                        sh 'npm test --if-present'
                    } else {
                        echo 'No tests executed: unknown project type'
                    }
                }
            }
            post {
                always {
                    junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml, **/build/test-results/**/*.xml'
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: '**/target/*.jar, **/build/libs/*.jar, dist/**, **/*.war', allowEmptyArchive: true
            }
        }

        stage('Docker Build & Push') {
            when { allOf { expression { fileExists('Dockerfile') }; expression { return params.PUSH_DOCKER } } }
            steps {
                script {
                    def tag = "${params.DOCKER_IMAGE}:${env.BUILD_TAG}"
                    sh "docker build -t ${tag} ."
                    // If you have Jenkins credentials for registry, use withCredentials + docker login here.
                    sh "docker push ${tag}"
                    echo "Pushed ${tag}"
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
            cleanWs()
        }
    }
}