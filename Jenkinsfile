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
        stage('commit version update') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'github', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
                        
                        sh 'git config --global user.email "presidentsanya@gmail.com"'
                        sh 'git config --global user.name "ojaysanya"'
                        sh 'git init --initial-branch=main'
                        sh 'git remote set-url origin git@gitlab.com:ojaysanya/jenkins-simple-profile-app.git'
                        
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push -u origin HEAD:main'
                    }
                }
            }
        }
    }
}
