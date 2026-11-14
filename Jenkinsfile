
pipeline {
    agent any

    environment {
        IMAGE = 'almoudir/tp4:latest'
    }

    stages {
        stage('Cloning Git') {
            steps {
                echo 'Source code retrieved from GitHub'
            }
        }

        stage('Building Image') {
            steps {
                sh 'docker build -t "$IMAGE" .'
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    docker image inspect "$IMAGE" > /dev/null
                    test -f index.html
                    grep -q "Welcome BDCC" index.html
                    echo "Tests passed!"
                '''
            }
        }

        stage('Publish Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        set -eu
                        export DOCKER_CONFIG="$(mktemp -d)"
                        trap 'rm -rf "$DOCKER_CONFIG"' EXIT

                        printf '%s' "$DOCKERHUB_TOKEN" |
                            docker login \
                                --username "$DOCKERHUB_USER" \
                                --password-stdin

                        docker push "$IMAGE"
                    '''
                }
            }
	
        }
	
	stage('Deploy') {
    steps {
        sh '''
            set -e

            echo "=== Deploying application ==="

            docker rm -f tp4-container 2>/dev/null || true

            docker run -d \
                --name tp4-container \
                --restart unless-stopped \
                -p 8081:80 \
                almoudir/tp4:latest

            echo "=== Verifying deployment ==="
            docker ps --filter "name=tp4-container"
            curl --fail --retry 5 --retry-delay 2 \
                http://127.0.0.1:8081/

            echo "=== Deployment successful ==="
        '''
    }
}

    }
}
