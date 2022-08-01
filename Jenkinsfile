pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    dir("app") {
                        sh 'npm version minor'
                        def version = sh (returnStdout: true, script: "grep 'version' package.json | cut -d '\"' -f4 | tr '\\n' '\\0'")
                        env.IMAGE_NAME = "$BUILD_NUMBER"

                    }
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    dir("app") {
                        
                        sh "npm install"
                        
                    } 
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-repo', usernameVariable: 'USER', passwordVariable: 'PWD')]){
                    sh "docker build -t ojaysanya/myapp:${IMAGE_NAME} ."
                    sh "echo ${PWD} | docker login -u ${USER} --password-stdin"
                    sh "docker push ojaysanya/myapp:${IMAGE_NAME}"
                }
            }
        }
        stage('Deploy to EC2 Instance') {
            steps {
                script {
                    sh "docker-compose -f docker-compose.yaml up --detach"
                }
            }
        }
    }
}
