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

        stage('SonarQube Coverage') {
            steps {
                script {
                    def response = httpRequest(
                        url: "http://10.131.103.92:9000/api/measures/component?component=Task1&metricKeys=coverage",
                        authentication: 'SONARQUBE_TOKEN'
                    )
                    def json = readJSON text: response.content
                    def coverage = json.component.measures[0].value
                    echo "Code Coverage from SonarQube: ${coverage}%"
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'JFROG', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        def timestamp = new Date().format("yyyy-MM-dd HH:mm:ss", TimeZone.getTimeZone('IST'))
                        echo "Artifact uploaded at: ${timestamp}"
                        sh '''
                            FILE_NAME=$(basename dist/*.jar)
                            curl -u $USERNAME:$PASSWORD -T dist/$FILE_NAME "$ARTIFACTORY_URL/$ARTIFACTORY_REPO/Task2/$FILE_NAME"
                        '''
                    }
                }
            }
        }
    }
}
