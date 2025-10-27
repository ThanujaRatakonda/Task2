pipeline {
    agent any

    tools {
        jdk 'Java'
        ant 'ant'
    }

    environment {
        ARTIFACTORY_URL = 'http://10.131.103.92:8081/artifactory'
        ARTIFACTORY_REPO = 'libs-release-local'
        SONAR_HOST = 'http://10.131.103.92:9000'
        SONAR_PROJECT_KEY = 'Task1'
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
                        url: 'https://github.com/ThanujaRatakonda/Task1.git',
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

        stage('Show SonarQube Coverage') {
            steps {
                withCredentials([string(credentialsId: 'SonarQube', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        echo "Fetching SonarQube coverage..."
                        curl -s -u $SONAR_TOKEN: "$SONAR_HOST/api/measures/component?component=$SONAR_PROJECT_KEY&metricKeys=coverage"
                    '''
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'JFROG', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        def uploadTime = new Date().format("yyyy-MM-dd HH:mm:ss", TimeZone.getTimeZone('IST'))
                        echo "Artifact uploaded at: ${uploadTime}"
                        sh '''
                            FILE_NAME=$(basename dist/*.jar)
                            curl -u $USERNAME:$PASSWORD -T dist/$FILE_NAME "$ARTIFACTORY_URL/$ARTIFACTORY_REPO/Task1/$FILE_NAME"
                        '''
                    }
                }
            }
        }
    }
}
