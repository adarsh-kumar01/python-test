pipeline {
    agent {
        kubernetes {
            yaml '''
                apiVersion: v1
                kind: Pod
                spec:
                  containers:
                  - name: python
                    image: python:3.9-slim
                    command:
                    - cat
                    tty: true
                    volumeMounts:
                    - name: python-cache
                      mountPath: /root/.cache/pip
                  - name: kaniko
                    image: gcr.io/kaniko-project/executor:debug
                    command:
                    - /busybox/cat
                    tty: true
                  volumes:
                  - name: python-cache
                    persistentVolumeClaim:
            '''
        }
    }

    environment {
        DOCKER_HUB_REPO = "adarsh321/kaniko"
        IMAGE_TAG = "3.0.0"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Python Dependencies') {
            steps {
                container('python') {
                    sh 'pip install --upgrade pip'
                    sh 'pip install -r requirements.txt'  // Installs Python dependencies if a requirements.txt exists
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                container('kaniko') {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_HUB_USR', passwordVariable: 'DOCKER_HUB_PSW')]) {
                    script {
                            sh """
                                echo '{"auths":{"https://index.docker.io/v1/":{"auth":"'"\$(echo -n ${DOCKER_HUB_USR}:${DOCKER_HUB_PSW} | base64)"'"}}}' > /kaniko/.docker/config.json
                                /kaniko/executor --dockerfile="/Dockerfile" --context `pwd` --destination ${DOCKER_HUB_REPO}:${IMAGE_TAG}
                            """
                        }
                    }
                }
            }
        }
    }
}
