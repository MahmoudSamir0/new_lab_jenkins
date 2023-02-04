pipeline {
    agent { label 'jenkins-agent' }
    stages {
        stage('build') {
            steps {
                script {
                   if (env.BRANCH_NAME == "build") {
                       withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                           sh """
                                docker login -u $USERNAME -p $PASSWORD
                                docker build -t 2534m/jenkins-pro:${BUILD_NUMBER} .
                                docker push 2534m/jenkins-pro:${BUILD_NUMBER}
                                echo ${BUILD_NUMBER} > ../jenkins-pro.txt
                           """
                       }
                    }
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    if (env.BRANCH_NAME == "dev" || params.ENV == "test" || params.ENV == "prod") {
                            withCredentials([file(credentialsId: 'kubernetes-con', variable: 'KUBECONFIG')]) {
                          sh """
                              export BUILD_NUMBER=\$(cat ../jenkins-pro.txt)
                              mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                              cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                              rm -f Deployment/deploy.yaml.tmp
                              kubectl apply -f Deployment --kubeconfig=${KUBECONFIG}
                            """
                        }
                    }
                }
            }
        }
    }
}
