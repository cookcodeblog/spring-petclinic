def mvnCmd = "mvn -s nexus-settings.xml -U -B"
pipeline {
    agent {
        abel "maven"
    }

    environment {
        APP_GIT_REPO = "https://github.com/cookcodeblog/spring-petclinic.git"
        APP_GIT_BRANCH = ""
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '1'))
        timeout(time: 15, unit: 'MINUTES')
        ansiColor('xterm')
        timestamps()
    }

    stages {
        stage('Build App') {
            steps {
                git branch: "${APP_GIT_BRANCH}", url: "${APP_GIT_REPO}"
                sh "${mvnCmd} clean install -DskipTests=true"
            }
        }
    }
}