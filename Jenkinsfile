pipeline {

    agent any

    environment {

        JFROG_URL = "http://4.240.17.188:8082/artifactory"
        ARTIFACT_NAME = "artifact.txt"
    }

    stages {

        stage('Checkout Code') {

            steps {

                git branch: 'main',
                url: 'https://github.com/amithps1996/jenkins-CK.git'
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

                    ls -R
                '''

                stash includes: 'output/**', name: 'build-artifact'
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

                unstash 'build-artifact'

                withCredentials([
                    usernamePassword(
                        credentialsId: '9a34a1f4-536a-45ac-84e3-3c76e1212b31',
                        usernameVariable: 'JF_USER',
                        passwordVariable: 'JF_PASS'
                    )
                ]) {

                    sh '''
                        echo "Checking artifact"

                        ls -R

                        curl -v -u $JF_USER:$JF_PASS \
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
                        echo "Deploying application"

                        kubectl apply -f deployment.yaml

                        echo "Checking pods"

                        kubectl get pods
                    '''
                }
            }
        }

        stage('Verification') {

            steps {

                input message: 'Approve deployment cleanup?'

                echo "Deployment verified successfully"
            }
        }

        stage('Destroy Deployment') {

            steps {

                sh '''
                    echo "Deleting deployment"

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
