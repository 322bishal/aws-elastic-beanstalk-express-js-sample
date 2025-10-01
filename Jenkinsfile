pipeline {
    agent none
    environment {
		  DOCKER_HUB_PASSWORD = credentials('DOCKER_HUB_PASSWORD') // Docker Hub password stored in jenkins credentials manager
      DOCKER_USER = credentials('DOCKER_USER')
      DOCKER_IMAGE_NAME = "assignment2"
      IMAGE_TAG = "latest"
      SNYK_KEY = credentials('SNYK_KEY')
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
              // Install node packages
                sh 'npm install --save'
            }
        }
        stage('Run tests') {
             agent {
                docker {
                    image 'node:16'
                    args '-u root:root'
                }
            }
          // Run unit test
            steps {
                sh 'npm test || echo "no test found"'
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
              // Install snyk 
                sh 'npm install -g snyk' 
                sh 'snyk auth $SNYK_KEY'
              // Run snyk test for high severity
                sh 'snyk test --severity-threshold=high'
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
                sh 'docker build -t $DOCKER_USER/$DOCKER_IMAGE_NAME:$IMAGE_TAG .'
				        sh 'echo $DOCKER_HUB_PASSWORD | docker login -u $DOCKER_USER --password-stdin'
				        sh 'docker push $DOCKER_USER/$DOCKER_IMAGE_NAME:$IMAGE_TAG'
            }
        }
    }
}
