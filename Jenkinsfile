pipeline {
    agent any
    environment{
        DOCKER_TAG = getDockerTag()
    }
    stages{
        stage('Build Docker Image'){
            steps{
                sh "docker build . -t anilkumblepuli/vprofile:$BUILD_NUMBER "
            }
        }
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'docker-hub', variable: 'dockerHubPwd')]) {
                    sh "docker login -u anilkumblepuli -p Anilkumar@123"
                    sh "docker push anilkumblepuli/vprofile:$BUILD_NUMBER"
                }
            }
        }
        stage('Deploy to k8s'){
            steps{
                sh "chmod +x changeTag.sh"
                sh "./changeTag.sh $BUILD_NUMBER"
                sshagent(['kops-machine']) {
                    sh "scp -o StrictHostKeyChecking=no services.yml node-app-pod.yml ec2-user@52.66.70.61:/home/ec2-user/"
                    script{
                        try{
                            sh "ssh ubuntu@13.233.164.75 kubectl apply -f ."
                        }catch(error){
                            sh "ubuntu@13.233.164.75 kubectl create -f ."
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
