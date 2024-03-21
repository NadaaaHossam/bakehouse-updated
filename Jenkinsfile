pipeline {
    agent { label 'java11' }
    stages {
        stage('build') {
            when {
                anyOf { branch 'dev'; branch 'test'; branch 'release' }
            }
            steps {
                echo 'build'
                        withCredentials([usernamePassword(credentialsId: 'docker-nada', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                            sh '''
                                docker build -t nada/bakehouseiti:v${BUILD_NUMBER}-${BRANCH_NAME} .
                                docker login -u ${USERNAME} -p ${PASSWORD}
                                docker push nada/bakehouseiti:v${BUILD_NUMBER}-${BRANCH_NAME}
                            '''
                        }
            }
        }
        stage('deploy') {
            when {
                anyOf { branch 'dev'; branch 'test'; branch 'release' }
            }
            steps {
                echo 'deploy'
                        withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                            sh '''
                                cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                                mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                                kubectl apply -f Deployment --kubeconfig ${KUBECONFIG} -n ${BRANCH_NAME}
                            '''
                        }
                    }
            }
        }
    }
