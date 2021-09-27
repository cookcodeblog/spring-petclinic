def mvnCmd = "mvn -s nexus-settings.xml"
pipeline {
    agent {
        label "maven"
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
            steps {
                git branch: "${env.APP_GIT_BRANCH}", url: "${env.APP_GIT_REPO}"
                sh "${mvnCmd} clean install -DskipTests=true"
            }
        }
        stage('Test') {
            steps {
                sh "${mvnCmd} test"
                step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
            }
        }
         stage('Code Analysis') {
            steps {
                script {
                sh "${mvnCmd} install sonar:sonar -Dsonar.host.url=http://sonarqube-sonarqube.will-cicd-sonarqube:9000 -DskipTests=true"
                }
            }
        }
    }
   
}