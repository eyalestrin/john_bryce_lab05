pipeline {
    agent any
    parameters {
        string defaultValue: '300', name: 'INTERVAL'
        string defaultValue: 'us-east-1', name: 'REGION'
    }
    environment {
        AWS_CREDENTIALS = credentials('credentials')
        DOCKER_REGISTRY = "eyales/johnbryce"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_id')
        dockerImage = ''
    }
    stages {
        stage('Init Cleanup') {
            steps {
                echo "Cleaning old deployments of the application"
                cleanWs()
                sh "docker kill myapp || true"
                sh "docker rm myapp || true"
            }
        }
        stage('SCM Step') {
            steps {
                echo "Pulling code from GitHub"
                git url: 'https://github.com/eyalestrin/john_bryce_lab05.git', branch: 'master'
            }
        }
        stage('Build Step') {
            steps {
                echo "Creating an application from Dockerfile, with version that match the current build running number"
                sh "cat $AWS_CREDENTIALS | tee credentials"
                sleep 2
                script {
                    dockerImage = docker.build(DOCKER_REGISTRY + ":${currentBuild.number}.0","-f Dockerfile .")
                }
            }
        }
        stage('Deploy Step') {
            steps {
                echo "Running the docker container, with version that match the current build running number"
                sh "docker run -itd --log-driver=json-file --name myapp --env INTERVAL=${params.INTERVAL} --env REGION=${params.REGION} $DOCKER_REGISTRY:${currentBuild.number}.0"
            }
        }
        stage('Print Output Step') {
            steps {
                echo "Printing docker output"
                sleep 2
                sh "docker logs myapp"
            }
        }
        stage('DockerHub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin '
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin  ' 
            }
        }
        stage('Push image to DockerHub') { 
            steps {
                sh (script : """docker push $DOCKER_REGISTRY:${currentBuild.number}.0""", returnStdout: false)
            }
        }
        stage('Install Helm package and yq') {
            steps {
                sh 'curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | tee /usr/share/keyrings/helm.gpg > /dev/null'
                sh 'apt-get install apt-transport-https --yes'
                sh 'echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | tee /etc/apt/sources.list.d/helm-stable-debian.list'
                sh 'apt-get update'
                sh 'apt-get install helm'
                sh 'apt install wget'
                sh 'wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64'
//                sh 'wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/download/3.3.1/yq_linux_amd64'
                sh 'chmod a+x /usr/local/bin/yq'
            }
        }
        stage('Edit Helm chart') {
            steps {
                sh 'helm create myapp'
//                sh 'cd myapp'
                sh 'pwd'
//                sh 'echo $HOME'
                sh 'ls -lh ./myapp/values.yaml'
//                sh 'yq'
                sleep 2
                sh """'yq -i e '"'"'.image.repository = \"$DOCKER_REGISTRY\"'"'"' ./myapp/values.yaml'"""
//                sh """'yq r ./myapp/values.yaml '"'"'.image.repository = \"$DOCKER_REGISTRY\"'"'"' '"""
                sh """'yq -i '"'"'.image.tag = \\\"${currentBuild.number}.0\\\"'"'"' ./myapp/values.yaml'"""
                sleep 2
                sh 'cat values.yaml'
            }
        }
    }
}
