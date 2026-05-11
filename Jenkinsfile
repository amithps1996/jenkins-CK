pipeline {

    agent any

    environment {

        JFROG_URL = "http://YOUR_VM_IP:8081/artifactory"
        ARTIFACT_NAME = "artifact.txt"
    }

    stages {

        stage('Checkout Code') {

            steps {

                git branch: 'main',
                url: 'https://github.com/YOUR_USERNAME/jenkins-CK.git'
            }
        }

        stage('Build Inside Docker Container Slave') {

            agent {

                docker {

                    image 'python:3.10'
                    args '-u root'
                }
            }

            steps {

                sh '''
                    echo "Running inside Docker container slave"

                    python --version

                    mkdir -p output

                    echo "Build artifact generated" > output/artifact.txt
                '''
            }
        }

        stage('Use Dockerfile as Agent') {

            agent {

                dockerfile {

                    filename 'Dockerfile'
                }
            }

            steps {

                sh '''
                    echo "Running custom Docker image"

                    python app.py
                '''
            }
        }

        stage('Upload Artifact to JFrog') {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-creds',
                        usernameVariable: 'JF_USER',
                        passwordVariable: 'JF_PASS'
                    )
                ]) {

                    sh '''
                        curl -u $JF_USER:$JF_PASS \
                        -T output/artifact.txt \
                        $JFROG_URL/demo-repo/artifact.txt
                    '''
                }
            }
        }

        stage('Deploy Using Kubernetes Pod Agent') {

            agent {

                kubernetes {

                    label 'k8s-agent'

                    yaml """
apiVersion: v1
kind: Pod

spec:

  containers:

  - name: kubectl

    image: bitnami/kubectl

    command:
    - cat

    tty: true
"""
                }
            }

            steps {

                container('kubectl') {

                    sh '''
                        kubectl apply -f deployment.yaml

                        kubectl get pods
                    '''
                }
            }
        }

        stage('Verification') {

            steps {

                input message: 'Approve deployment cleanup?'

                echo "Deployment verified"
            }
        }

        stage('Destroy Deployment') {

            steps {

                sh '''
                    kubectl delete -f deployment.yaml
                '''
            }
        }
    }

    post {

        always {

            echo 'Pipeline execution completed'

            sh 'docker system prune -f'
        }

        success {

            echo 'Pipeline succeeded'
        }

        failure {

            echo 'Pipeline failed'
        }
    }
}
