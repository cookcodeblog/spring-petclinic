def mvnCmd = "mvn -s nexus-settings.xml"
pipeline {
    agent {
        label "maven"
    }

    environment {
        // define glolab vars
        //foo="bar"

        PIPELINES_NAMESPACE = "will-cicd-jenkins"

        // applicaiton info
        PROJECT_NAMESPACE = "will-petclinic"
        APP_NAME = "spring-petclinic"
        APP_BUILD_CONFIG = "${env.APP_NAME}"
        APP_DEPLOYMENT = "${env.APP_NAME}"
        APP_GIT_MAIN_BRANCH = "main"
        // TODO App version
        APP_VERSION = "0.0.1"
        
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '1'))
        timeout(time: 15, unit: 'MINUTES')
        ansiColor('xterm')
    }

    stages {
        stage('Prepare Environment') {
            steps {
                // https://e.printstacktrace.blog/jenkins-pipeline-environment-variables-the-definitive-guide/
                // https://{JENKINS_HOST}/env-vars.html/
                // Jenkins built-in env vars examples:
                // BRANCH_NAME=main
                // GIT_BRANCH=main
                // GIT_URL=https://github.com/cookcodeblog/spring-petclinic.git
                // BUILD_NUMBER=10
                sh 'printenv'

            }
        }
        stage('Build App') {
            steps {
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
        stage('Archive App') {
            steps {
                // need create `nx-deploy` Nexus role with `nx-repository-view-*-*-*` privileges
                // create `jenkins-user` Nexus user with this role
                // configure `jenkins-user` server login information for repositories and mirror in settings.xml
                // configure repository information in `distributionManagement` of pom.xml
                sh "${mvnCmd} deploy -DskipTests=true"
            }
        }
        stage('Build Image') {
            steps {
                sh "cp target/${env.APP_NAME}-*.jar target/app.jar"
                script {
                    openshift.withCluster() {
                        openshift.withProject("${env.PROJECT_NAMESPACE}") {
                            // start a build for selected openshift build config
                            openshift.selector("bc", "${env.APP_BUILD_CONFIG}").startBuild("--from-file=target/app.jar", "--wait=true")
                            // tag image
                            openshift.tag("${env.APP_NAME}:latest", "${env.APP_NAME}:${env.APP_VERSION}")
                            // TODO push image to external image registry
                            // TODO git tag
                        }
                    }
                }
            }
        }
        stage('Deploy Image') {
            steps {}
                openshift.withCluster() {
                    openshift.withProject("${env.PROJECT_NAMESPACE}") {
                        openshift.selector("deployment", "${env.APP_DEPLOYMENT}").rollout().latest()
                    }
                }
            }
        }
    }
   
}