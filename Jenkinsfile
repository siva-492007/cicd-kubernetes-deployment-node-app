pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t vigneshsiva/cicd-kubenetes-deployment-node-app:${DOCKER_TAG}"
            }
        }
        stage('Nexus Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub-pwd', variable: 'DOCKER_HUB_PASSWORD')]) {
                    sh "docker login -u vigneshsiva -p ${DOCKER_HUB_PASSWORD}"
                }
                sh "docker push vigneshsiva/cicd-kubenetes-deployment-node-app:${DOCKER_TAG}"
            }
        }
        stage('K8 Deploy'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh ${DOCKER_TAG}"
                sshagent(['kubeadm-master']) {
                    sh "scp -o StrictHostKeyChecking=no service.yml node-app-pod.yml ubuntu@3.85.112.164:/home/ubuntu/"
                    script{
                        try{
                            sh "ssh ubuntu@3.85.112.164 kubectl apply -f ."
                        }
                        catch(error){
                            sh "ssh ubuntu@3.85.112.164 kubectl create -f ."
                        }
                    }
                }
            }
        }
    }
}

def getDockerTag(){
    def tag  = sh script: 'git rev-parse HEAD', returnStdout: true
    return tag
}
