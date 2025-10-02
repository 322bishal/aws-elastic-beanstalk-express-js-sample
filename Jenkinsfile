pipeline {
    agent none
    environment {
        DOCKER_HUB_PASSWORD = credentials('DOCKER_HUB_PASSWORD') // Docker Hub password stored in Jenkins credentials manager
        DOCKER_USER = credentials('DOCKER_USER')
        DOCKER_IMAGE_NAME = "assignment2"
        IMAGE_TAG = "latest"
        SNYK_KEY = credentials('SNYK_KEY')
    }
    
    options {
        buildDiscarder(logRotator(daysToKeepStr: '30', numToKeepStr: '30')) // Retain logs for 30 days / 30 builds
        timestamps() // Add timestamps to console logs
    }

    stages {
        stage('npm install') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                echo "=== Installing NodeJS Dependencies ==="
                sh 'npm install --save | tee npm-install.log'
            }
        }
        
        stage('Run tests') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                echo "=== Running Unit Tests ==="
                sh '''
                  if [ -f package.json ] && grep -q '"test"' package.json; then
                    npm test | tee unit-tests.log
                  else
                    echo "no test found" | tee unit-tests.log
                  fi
                '''
            }
        }

        stage('Snyk Test Vulnerabilities') {
            agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
            steps {
                echo "=== Running Security Scan with Snyk ==="
                sh 'npm install -g snyk | tee snyk-install.log'
                sh 'snyk auth $SNYK_KEY | tee snyk-auth.log'
                sh 'snyk test --severity-threshold=high | tee snyk-test.log'
            }
        }
        
        stage('Build and push docker image') {
            agent {
                docker {
                    image 'docker:24'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                echo "=== Building Docker Image ==="
                sh 'docker build -t $DOCKER_USER/$DOCKER_IMAGE_NAME:$IMAGE_TAG . 2>&1 | tee docker-build.log'
                echo "=== Logging into Docker Hub ==="
                sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_USER --password-stdin 2>&1 | tee docker-login.log'
                echo "=== Pushing Docker Image ==="
                sh 'docker push $DOCKER_USER/$DOCKER_IMAGE_NAME:$IMAGE_TAG 2>&1 | tee docker-push.log'
            }
        }
    }

    post {
        always {
            echo "=== Build Complete: Archiving all logs ==="
            script {
                node('master') {
                    // Archive all logs from all stages
                    archiveArtifacts artifacts: '*.log', allowEmptyArchive: true
                }
            }
        }
    }
}
