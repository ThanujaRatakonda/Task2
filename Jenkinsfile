pipeline {
    agent any

    tools {
        jdk 'Java'
        ant 'ant'
    }

    environment {
        ARTIFACTORY_URL = 'http://10.131.103.92:8081/artifactory'
        ARTIFACTORY_REPO = 'Task2'
    }

    stages {
        stage('Cleanup') {
            steps {
                deleteDir()
            }
        }

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    userRemoteConfigs: [[
                        url: 'https://github.com/ThanujaRatakonda/Task2.git',
                        credentialsId: 'GITHUB'
                    ]],
                    branches: [[name: '*/master']]
                ])
            }
        }

        stage('Build') {
            steps {
                sh 'ant clean dist'
            }
        }
               stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh 'sonar-scanner'
                }
            }
        }           
        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'JFROG', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    sh '''
                        FILE_NAME=$(basename dist/*.jar)
                        curl -u $USERNAME:$PASSWORD -T dist/$FILE_NAME "$ARTIFACTORY_URL/$ARTIFACTORY_REPO/Task2/$FILE_NAME"
                    '''
                }
            }
        }
    }
}

