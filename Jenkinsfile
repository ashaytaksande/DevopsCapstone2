pipeline {
    agent any

    environment {
        DockerCred = credentials('dockercred')
        version = "${env.BUILD_ID}-${env.GIT_COMMIT.substring(0, 5)}"
        branchName = "${env.GIT_BRANCH.split('/').size() == 1 ? env.GIT_BRANCH.split('/')[-1] : env.GIT_BRANCH.split('/')[1..-1].join('/')}"
    }

    stages {
        stage('Build when new code is pushed to the master branch.') {
            agent {
                label 'K8sMaster'
            }
            when {
                expression {
                    branchName == 'master'
                }
            }
            steps {
                sh '''
                    sudo docker build -t ashayalmighty/website:${version} .
                '''
            }
        }


        stage('Push image to Docker hub') {
            agent {
                label 'K8sMaster'
            }

            when {
                expression {
                    branchName == 'master'
                }
            }

            steps {
                sh '''
                    echo $DockerCred_PSW | sudo docker login -u $DockerCred_USR --password-stdin
                    sudo docker push ashayalmighty/website:${version}
                '''
            }
        }

        stage('Deploy the containerized code from Docker hub with 2 replicas.') {
            agent {
                label 'K8sMaster'
            }

            when {
                expression {
                    branchName == 'master'
                }
            }

            steps {
                sh '''
                    kubectl apply -f website.yaml
                    kubectl apply -f node.yaml
                '''
            }
        }

    }
}
