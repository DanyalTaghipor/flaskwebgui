pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
    - name: k8s
      image: alpine/k8s:1.25.13
      imagePullPolicy: IfNotPresent
      command:
        - sleep
      args:
        - infinity
      volumeMounts:
        - name: docker-socket
          mountPath: /var/

    - name: docker
      image: docker:24.0.6
      imagePullPolicy: IfNotPresent
      readinessProbe:
        exec:
          command: [sh, -c, "ls -S /var/run/docker.sock"]
      command:
        - sleep
      args:
        - 99d
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run

    - name: docker-daemon
      image: docker:24.0.6-dind
      securityContext:
        privileged: true
      volumeMounts:
        - name: docker-socket
          mountPath: /var/run
  volumes:
    - name: docker-socket
      emptyDir: {}
"""
        }
    }

    environment {
        DOCKER_IMAGE = "danytgh/webgui"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/DanyalTaghipor/webgui', branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                container('docker') {
                    script {
                        dockerImage = docker.build("${DOCKER_IMAGE}:${env.BUILD_ID}")
                    }
                }
            }
        }
        stage('Push Docker Image') {
            steps {
                container('docker') {
                    script {
                        docker.withRegistry('https://registry.hub.docker.com', 'docker') {
                            dockerImage.push("${env.BUILD_ID}")
                            dockerImage.push("latest")
                        }
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                container('k8s') {
                    script {
                        sh 'helm upgrade --install webgui helm/ --values helm/my-values.yaml'
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
