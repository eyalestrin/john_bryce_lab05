pipeline {
    agent any
    parameters {
        string defaultValue: '300', name: 'INTERVAL'
        string defaultValue: 'us-east-1', name: 'REGION'
//        String currentJobName = currentBuild.projectName
//        String paramValue = getParamValue(currentJobName)
    }
    environment {
        AWS_CREDENTIALS = credentials('credentials')
        DOCKER_REGISTRY = "eyales/johnbryce"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub_id')
        dockerImage = ''
        GITHUB_CREDENTIALS = credentials('github')
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
                git url: 'https://github.com/eyalestrin/john_bryce_lab05.git', branch: 'dev'
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
        stage('Docker Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin '
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin  ' 
            }
        }
        stage('Push image to DockerHub') { 
            steps {
                sh (script : """docker push $DOCKER_REGISTRY:${currentBuild.number}.0""", returnStdout: false)
		sleep 2
		sh 'docker logout'
            }
        }
        stage('Install yq package') {
            steps {
                sh 'apt install wget'
                sh 'wget -qO /usr/local/bin/yq https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64'
                sh 'chmod a+x /usr/local/bin/yq'
            }
        }
        stage('Edit Helm chart') {
            steps {
//                sh 'pwd'
//                sh 'echo $HOME'
//                sh 'echo ${currentBuild.projectName}'
                dir('/home/jenkins/workspace/john_bryce_lab05/myapp-helm/') {
                sh (script : """ yq -i \'.image.repository = \"$DOCKER_REGISTRY\"\' values.yaml """, returnStdout: false)
                sh (script : """ yq -i \'.image.tag = \"${currentBuild.number}.0\"\' values.yaml """, returnStdout: false)
//                sh (script : """ cat values.yaml | grep repository """)
//                sh (script : """ cat values.yaml | grep tag: """)
                }
            }
        }

        stage('Git Push to Master') {
            steps {
            	script {
                	withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                		sh 'git commit -am '"'"'Updating Docker version ${currentBuild.number}.0'"'"''
                		sh 'git push origin master'
			}
		}
           }
	}
//    }
//}

//    }
//	post {
//        always {
//		    sh 'docker logout'
//		}

//}
}
}
