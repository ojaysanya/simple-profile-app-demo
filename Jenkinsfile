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
                        # update application version in the package.json file with one of these release types: patch, minor or major
                        # this will commit the version update
                        npm version minor

                        # read the updated version from the package.json file
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version

                        # set the new version as part of IMAGE_NAME
                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    }

                    # alternative solution without Pipeline Utility Steps plugin: 
                    # def version = sh (returnStdout: true, script: "grep 'version' package.json | cut -d '\"' -f4 | tr '\\n' '\\0'")
                    # env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    dir("app") {
                        # install all dependencies needed for running tests
                        sh "npm install"
                        sh "npm run test"
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
                    withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', usernameVariable: 'USER', passwordVariable: 'PWD')]) {
                        # git config here for the first time run
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'

                        sh "git remote set-url origin https://${USER}:${PWD}@gitlab.com/devops-bootcamp3/node-project.git"
                        sh 'git add .'
                        sh 'git commit -m "ci: version bump"'
                        sh 'git push origin HEAD:jenkins-jobs'
                    }
                }
            }
        }
    }
}
