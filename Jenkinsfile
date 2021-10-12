pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '20'))
        disableConcurrentBuilds()
        timeout (time: 10, unit: 'MINUTES')
        timestamps()
    }
    parameters {
        string(defaultValue: "https://github.com/leonardtia1/maven-project.git", description: 'supply a github repository', name: 'github')
        string(defaultValue: "master", description: 'supply the branch for the docker image', name: 'branch')
        string(defaultValue: "develop", description: 'supply a tag ', name: 'tag')
    }
    environment {
        registry = 'leonardtia/devops-test-repo'
        registryCredential = 'Docker-Hub-Credentials'
        dockerImage = ''
    }
    stages {
        stage ('checkout') {
            agent {
                docker { image 'bitnami/git' }
            }
            steps {
                dir("${WORKSPACE}/build") {
                    checkout([
                        $class: 'GitSCM',
                        branches: [[name: "*/master"]],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [[$class: 'LocalBranch']],
                        submoduleCfg: [],
                        userRemoteConfigs: [[
                        url: 'https://github.com/leonardtia1/maven-project.git',
                        credentialsId: 'GitHub-Credentials'
                        ]]
                    ])
                }
            }
        }
        stage('Building the code') {
            agent {
                docker { image 'maven:3.8.1-adoptopenjdk-11' }
            }
            steps {
                dir("${env.WORKSPACE}/build") {
                    sh '''
                    ls 
                    pwd
                    mvn --version
                    mvn clean install package
                    '''
                }
            }
        }
        stage ('Building the image') {
            agent {
                label 'master'
            }
            steps {
                script {
                    dir("${env.WORKSPACE}/build") {
                        docker.withRegistry('','Docker-Hub-Credentials' ) {
                            dockerImage = docker.build('leonardtia/devops-test-repo'+":${tag}")
                        }
                    }

                } 
            }
        }
        stage ('Pushing the image') {
            agent {
                label 'master'
            }
            steps {
                script {
                    dir("${env.WORKSPACE}/build") {
                        docker.withRegistry('','Docker-Hub-Credentials' ) {
                            dockerImage.push()
                        }
                    }
                } 
            }
        }
        stage ('Deployment') {
           agent {
                label 'master'
            }
            steps {
                sh '''
                sudo docker run -d -P ${registry}:${tag} 
                sudo docker ps   
                '''
            }
        }
    }
}
