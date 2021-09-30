pipeline {
    agent {
        label 'maven'
    }

    environment {
        // define glolab vars
        MVN_CMD = "mvn -s nexus-settings.xml"

        PIPELINES_NAMESPACE = "will-cicd-jenkins"
        

        // applicaiton info
        PROJECT_NAMESPACE = "will-petclinic"
        APP_NAME = "spring-petclinic"
        APP_BUILD_CONFIG = "${env.APP_NAME}"
        APP_DEPLOYMENT = "${env.APP_NAME}"
        IMAGE_REGISTRY = "image-registry.openshift-image-registry.svc:5000/${env.PROJECT_NAMESPACE}/"
        APP_IMAGE = "${env.APP_NAME}"
        APP_GIT_MAIN_BRANCH = "main"
        // TODO App version
        APP_VERSION = "0.0.1"
        
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '50', artifactNumToKeepStr: '1'))
        ansiColor('xterm')
        // the whole pipeline timeout
        // also recommend to set tiemout at each stage
        timeout(time: 20, unit: 'MINUTES')
    }

    stages {
        stage('Prepare Environment') {
            options {
                timeout(time: 1, unit: 'MINUTES')
            }
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
            options {
                timeout(time: 3, unit: 'MINUTES')
            }
            steps {
                sh "${env.MVN_CMD} clean install -DskipTests=true"
            }
        }
        stage('Test') {
            options {
                timeout(time: 3, unit: 'MINUTES')
            }
            steps {
                sh "${env.MVN_CMD} test"
                step([$class: 'JUnitResultArchiver', testResults: '**/target/surefire-reports/TEST-*.xml'])
            }
        }
        stage('Code Analysis') {
            options {
                timeout(time: 3, unit: 'MINUTES')
            }
            steps {
                script {
                    // need install SonarQube Scanner plugin and configure SonarQube server in Jenkins
                    // -Dsonar.host.url=http://sonarqube-sonarqube.will-cicd-sonarqube:9000
                    withSonarQubeEnv('sonarqube') {
                        sh "${env.MVN_CMD} install sonar:sonar -DskipTests=true"
                    }
                }
            }
        }
        stage('Archive App') {
            options {
                timeout(time: 2, unit: 'MINUTES')
            }
            steps {
                // need create `nx-deploy` Nexus role with `nx-repository-view-*-*-*` privileges
                // create `jenkins-user` Nexus user with this role
                // configure `jenkins-user` server login information for repositories and mirror in settings.xml
                // configure repository information in `distributionManagement` of pom.xml
                sh "${env.MVN_CMD} deploy -DskipTests=true"
            }
        }
        stage('Build Image') {
            options {
                timeout(time: 2, unit: 'MINUTES')
            }
            steps {
                sh "cp target/${env.APP_NAME}-*.jar target/app.jar"
                script {
                    openshift.withCluster() {
                        openshift.withProject("${env.PROJECT_NAMESPACE}") {
                            // start a build for selected openshift build config
                            // --from-file='': A file to use as the binary input for the build; 
                            // example a pom.xml or Dockerfile. Will be the only file in the build source
                            // wait until the build complete
                            openshift.selector("bc", "${env.APP_BUILD_CONFIG}").startBuild("--from-file=target/app.jar", "-w")
                            // tag image
                            // run `oc get imagestream` and `oc get imagestreamtag` to check image tags
                            openshift.tag("${env.APP_IMAGE}:latest", "${env.APP_IMAGE}:${env.APP_VERSION}")
                            // TODO push image to external image registry
                            // TODO git tag
                        }
                    }
                }
            }
        }
        stage('Deploy Image') {
            options {
                timeout(time: 1, unit: 'MINUTES')
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject("${env.PROJECT_NAMESPACE}") {
                            // Only DeploymentConfig can use rollout().latest()
                            // Use `oc set iamge deployment` to update image vesion inside deployment
                            // Nee set image change trigger for deployment
                            sh """
                                oc set image deployment ${env.APP_DEPLOYMENT} ${env.APP_IMAGE}=${env.IMAGE_REGISTRY}/${env.APP_IMAGE}:${env.APP_VERSION} -n ${env.PROJECT_NAMESPACE}
                            """
                        }
                    }
                }
            }
        }
        stage('Smoke Test') {
            options {
                timeout(time: 10, unit: 'MINUTES')
            }
            steps {
                script {
                    openshift.withCluster() {
                        openshift.withProject("${env.PROJECT_NAMESPACE}") {
                            // watch until the rollout complete
                            openshift.selector("deployment", "${env.APP_DEPLOYMENT}").rollout().status("-w")
                        }
                    }
                }
            }
        }
    }
   
}