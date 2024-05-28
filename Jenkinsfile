pipeline{
    agent any

    triggers {
        githubPush()
    }

    // tools{
    //     maven 'maven_3_5_0'
    // }

    // environment {
	// 	DOCKERHUB_CREDENTIALS=credentials('dockerpasswd')
    //     GIT_REPO_URL = "https://github.com/akshayraina999/django-to-do.git"
    //     DOCKER_IMAGE_NAME = "devops-poc"
	// } 

    // def latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim()

    stages {
        stage ("Clone devops files") {
            steps {
                echo "*************** cloning devops files *******************"
                git url: 'https://github.com/akshayraina999/asato-devops-poc.git', branch: 'main'
            }
        }

        // stage("Initialize") {
        //     steps {
        //         script {
        //             echo "*************** Initializing *******************"
        //             checkout([$class: 'GitSCM', userRemoteConfigs: [[url: env.GIT_REPO_URL]], branches: [[name: '*/main']]])
        //             sh "git fetch --tags"
        //             env.latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim()
        //             echo "Latest tag: ${env.latestTag}"
        //         }
        //     }
        // }

        stage ("Set environment variables") {
            steps {
                script {
                    echo "*************** Reading config *******************"
                    def config = readYaml file: 'config.yaml'
                    env.GIT_REPO_URL = config.GIT_REPO_URL
                    echo "${env.GIT_REPO_URL}"
                    env.DOCKER_IMAGE_NAME = config.DOCKER_IMAGE_NAME

                    checkout([$class: 'GitSCM', userRemoteConfigs: [[url: config.GIT_REPO_URL]], branches: [[name: '*/main']]])
                    sh "git fetch --tags"
                    env.latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim()
                    echo "Latest tag: ${env.latestTag}"
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

        // stage("Compile"){
        //     steps {
        //         echo "*************** compile the code ******************"
        //         sh 'mvn clean install'
        //     }
        // }

        stage ("Build docker image") {
            when { tag "dev-*" }
            steps {
                script {
                    echo "*************** Building docker image *******************"
                    // git url: 'https://github.com/akshayraina999/website-k8.git', branch: 'master'
                    def latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim()  
                    sh "docker build -t ${env.DOCKER_IMAGE_NAME}:${latestTag} ."
                }
            }
        }

        stage ("Push image to dockerhub") {
            steps {
                script {
                    echo "**************** Pushing docker image to the registery ************************"
                    def latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim()  
                    echo "2"
                    sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
                    echo "3"
                    echo "${env.DOCKER_IMAGE_NAME}:${latestTag}"
                    echo "4"
                    sh "docker tag ${env.DOCKER_IMAGE_NAME}:${latestTag} akshayraina/${env.DOCKER_IMAGE_NAME}:${latestTag}"
                    sh "docker push akshayraina/${env.DOCKER_IMAGE_NAME}:${latestTag}"
                    // sh 'docker rmi akshayraina/$JOB_NAME:v1.$BUILD_ID'
                }
            }
        }

        stage ("Deploying on kubernetes") {
            steps {
                sshagent (['kubernetes-server']){
                    git url: 'https://github.com/akshayraina999/asato-devops-poc.git', branch: 'main'
                    // def latestTag = sh(returnStdout: true, script: 'git describe --tags `git rev-list --tags --max-count=1`').trim() 
                    sh "sed -i 's/image_name/${env.DOCKER_IMAGE_NAME}/' deployment.yaml"
                    sh "sed -i 's/tag/${latestTag}/' deployment.yaml"
                    sh "ssh -o StrictHostKeyChecking=no azureuser@48.216.232.191 kubectl apply -f deployment.yaml --validate=false"
                }
            }
        }
    }
}