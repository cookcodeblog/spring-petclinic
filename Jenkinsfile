def mvnCmd = "mvn -s nexus-settings.xml"
pipeline {
    agent {
        label "maven"
    }

    environment {
        // define glolab vars
        foo="bar"
        
        //APP_GIT_REPO = "https://github.com/cookcodeblog/spring-petclinic.git"
        //APP_GIT_BRANCH = "main"
        
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '1'))
        timeout(time: 15, unit: 'MINUTES')
        ansiColor('xterm')
    }

    stages {
        stage('Prepare Environment') {
            steps {
                sh 'printenv'
            }
        }
        stage('Build App') {
            steps {
                // TODO: support multi branches
                //git branch: "${env.APP_GIT_BRANCH}", url: "${env.APP_GIT_REPO}"
                // git url: "${GIT_SOURCE_URL}", branch: "${GIT_SOURCE_REF}"
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
                    // need install SonarQube Scanner plugin and configure SonarQube server in Jenkins
                    // -Dsonar.host.url=http://sonarqube-sonarqube.will-cicd-sonarqube:9000
                    withSonarQubeEnv('sonarqube') {
                        sh "${mvnCmd} install sonar:sonar -DskipTests=true"
                    }
                }
            }
        }
        // stage('Archive App') {
        //     steps {
        //         sh "${mvnCmd} deploy -DskipTests=true"
        //     }
        // }
    }
   
}