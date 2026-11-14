
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
    }
}
