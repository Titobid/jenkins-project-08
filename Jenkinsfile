def gv

pipeline {
    agent any
    tools {
        nodejs 'NodeJS 24'
    }
    environment {
        DOCKER_REPOSITORY = 'titobid/jenkins-app'
    }

    stages {
        stage ("init") {
            steps {
                script{
                    gv = load "script.groovy"
                }
            }
        }
        stage ("increment version"){
            steps {
                dir('app'){
                    script {
                        echo 'incrementing app version ....'
                        sh 'npm version minor --no-git-tag-version'
                        env.APP_VERSION = sh(
                            script: "node -p \"require('./package.json').version\"",
                            returnStdout: true
                            ).trim()
                        env.IMAGE_TAG = "${env.APP_VERSION}-${env.BUILD_NUMBER}"
                        echo "Application version: ${env.APP_VERSION}"
                        echo "Docker image tag: ${env.IMAGE_TAG}"
                    }
                }
            }
        }
        stage ("run tests") {
            steps {
                dir('app'){
                    script {
                        echo 'Installing dependencies...'
                        sh 'npm install'
                        echo 'Running tests...'
                        sh 'npm test'
                    }
                }
            }
        }
        stage ("package nodejs app ") {
            steps {
                dir('app'){
                    script {
                        echo 'Packaging Node.js application...'
                        sh 'npm pack'
                    }
                }
            }
        }
        stage ("Build docker image") {
            steps {
                script{
                    echo 'Building docker image .... '
                    sh "docker build -t $DOCKER_REPOSITORY:$IMAGE_TAG ."
                }
            }
        }
         stage ("Pushing docker image") {
                    steps {
                        script{
                            echo 'Pushing docker image .... '
                            withCredentials([usernamePassword(credentialsId: 'docker-hub', passwordVariable: 'DOCKER_PASS', usernameVariable:'DOCKER_USER')]){
                            sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            sh "docker push $DOCKER_REPOSITORY:$IMAGE_TAG"
                            }
                        }
                    }
                }
        stage ("commiting version update") {
            steps{
                script {
                    withCredentials([usernamePassword(credentialsId: 'github-cred', passwordVariable: 'GIT_PASS', usernameVariable:'GIT_USER')]) {
                    sh 'git remote set-url origin https://${GIT_USER}:${GIT_PASS}@github.com/Titobid/jenkins-project-08.git'
                    sh 'git config --global user.email "jenkins@example.com" '
                    sh 'git config --global user.name "jenkins" '
                    sh 'git add app/package.json'
                    sh '''if [ -f app/package-lock.json ]; then
                          git add app/package-lock.json
                          fi'''
                    sh 'git commit -m "ci: bump version to $APP_VERSION"'
                    sh 'git push origin HEAD:main'
                    sh 'git status'
                    sh 'git branch'
                    sh 'git config --list'
                    }
                }
            }
        }
    }
}
