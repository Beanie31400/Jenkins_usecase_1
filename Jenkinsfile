def img
pipeline {
    environment {
        registry = "duchaido/my_first_project" //To push an image to Docker Hub, you must first name your local image using your Docker Hub username and the repository name that you created through Docker Hub on the web.
        registryCredential = 'Jenkins_credentials'
        githubCredential = 'GITHUB'
        dockerImage = ''
    }
    agent any
    stages {
        
        stage('checkout') {
                steps {
                    checkout([$class: 'GitSCM', branches: [[name: 'refs/remotes/origin/master']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'LocalBranch', localBranch:'**']], submoduleCfg: [], userRemoteConfigs: [[refspec:"+refs/pull/*:refs/remotes/origin/pr/*", refspec:"+refs/heads/*:refs/remotes/origin/*", url: 'https://github.com/Beanie31400/Jenkins_usecase_1.git']]])
                }
        }
        
        stage ('Test'){
                steps {
                sh 'pip install -r requirements.txt'
                sh "pytest testRoutes.py"
                }
        }
        
        stage ('Clean Up'){
            steps{
                sh returnStatus: true, script: 'docker stop $(docker ps -a | grep ${JOB_NAME} | awk \'{print $1}\') 2>/dev/null' 
                sh returnStatus: true, script: 'docker rmi $(docker images | grep ${registry} | awk \'{print $3}\') --force' //this will delete all images
                sh returnStatus: true, script: 'docker rm ${JOB_NAME} --force'
            }
        }

        stage('Build Image') {
            steps {
                script {
                    img = registry + ":${env.BUILD_ID}"
                    println ("${img}")
                    dockerImage = docker.build("${img}")
                }
            }
        }

        stage('Push To DockerHub') {
            steps {
                script {
                    withDockerRegistry([ credentialsId: "Jenkins_credentials", url: "" ]) {
                    dockerImage.push()
                    }
                }
            }
        }
                    
        stage('Deploy') {
           steps {
                sh label: '', script: "docker run -d --name ${JOB_NAME} -p 5000:5000 ${img}"
          }
        }

        }

    }