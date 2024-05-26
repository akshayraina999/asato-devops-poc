pipeline{
    agent any

    triggers {
        githubPush()
    }

    tools{
        maven 'maven_3_5_0'
    }

    environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerpasswd')
	} 

    stages {
        stage ("Read config file") {
            steps {
                script {
                    echo "*************** Reading config *******************"
                    def config = readYaml file: 'config.yaml'
                    env.GIT_REPO_URL = config.GIT_REPO_URL
                    echo "${env.GIT_REPO_URL}"
                    env.DOCKER_IMAGE_NAME = config.DOCKER_IMAGE_NAME
                }   
            }
        }

        stage ("Pull the code") {
            steps {
                script {
                    echo "*************** Checking out the latest code *******************"
                    checkout([$class: 'GitSCM', userRemoteConfigs: [[url: env.GIT_REPO_URL]], branches: [[name: '*/main']]])
                    sh "git fetch --tags"
                    def latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim()
                    echo "Latest tag: ${latestTag}"
                    sh "git checkout ${latestTag}"
                }
            }
        }

        stage("Compile"){
            steps {
                echo "*************** compile the code ******************"
                sh 'mvn clean install'
            }
        }

        stage ("Build docker image") {
            when { tag "dev-*" }
            steps {
                script {
                    echo "*************** Building docker image *******************"
                    def latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim()  
                    sh "docker build -t ${env.DOCKER_IMAGE_NAME}:${latestTag} ."
                }
            }
        }

        stage ("Push image to dockerhub") {
            steps {
                echo "**************** Pushing docker image to the registery ************************"
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                sh 'docker push akshayraina/$JOB_NAME:v1.$BUILD_ID'
                sh 'docker rmi akshayraina/$JOB_NAME:v1.$BUILD_ID'
            }
        }
    }
}