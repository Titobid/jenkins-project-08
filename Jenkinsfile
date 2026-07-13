library identifier: 'jenkins-shared-library@NodeJS-shared-library', retriever: modernSCM(
        [$class:'GitSCMSource',
         remote: 'https://github.com/Titobid/Jenkins-shared-library.git',
         credentialsId: 'github-cred'])

pipeline {
    agent any
    tools {
        nodejs 'NodeJS 24'
    }
    environment {
        DOCKER_REPOSITORY = 'titobid/jenkins-app'
    }
    stages {
        stage ("increment version"){
            steps {
                dir('app'){
                    script {
                        incrementVersion()
                    }
                }
            }
        }
        stage ("run tests") {
            steps {
                dir('app'){
                    script {
                       runTests()
                    }
                }
            }
        }
        stage ("package nodejs app ") {
            steps {
                dir('app'){
                    script {
                        packNodeApp()
                    }
                }
            }
        }
        stage ("Build and push image") {
            steps {
                script{
                    buildImage "$DOCKER_REPOSITORY:$IMAGE_TAG"
                    dockerLogin()
                    dockerPush "$DOCKER_REPOSITORY:$IMAGE_TAG"
                }
            }
        }

        stage ("commiting version update") {
            steps{
                script {
                   gitCommit()
                }
            }
        }
    }
}
