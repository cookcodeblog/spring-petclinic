def mvnCmd = "mvn -s nexus-settings.xml -U -B"
pipeline {
    agent {
        label "master"
    }

    environment {
        APP_GIT_REPO = "https://github.com/cookcodeblog/spring-petclinic.git"
        APP_GIT_BRANCH = "main"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '1'))
        timeout(time: 15, unit: 'MINUTES')
        ansiColor('xterm')
    }

    stages {
        stage('Build App') {
            agent {
                node {
                    label "jenkins-agent-maven"  
                }
            }
            steps {
                git branch: "${env.APP_GIT_BRANCH}", url: "${env.APP_GIT_REPO}"
                sh "${mvnCmd} clean install -DskipTests=true"
            }
        }
    }
}