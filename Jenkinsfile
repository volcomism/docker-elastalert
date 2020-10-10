pipeline {
    agent any

    parameters {
        booleanParam (name: 'DEPLOY_STAGE', defaultValue: env.BRANCH_NAME == 'master', description: 'deploy without approval')
    }

    environment {
      NAMESPACE = "volcomism"
      PROJECT = "elastalert"
      GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%h'", returnStdout: true)
      DEPLOY_STAGE = "${params.DEPLOY_STAGE}"
      SERVICE_NAME = "${env.GIT_URL.replaceAll('^.*/', '').replaceAll('.git$', '')}"
    }

    stages {
        stage('Build') {
            steps {
                sh "docker build -t ${env.REGISTRY_URL}/${env.NAMESPACE}/${env.PROJECT}:${GIT_COMMIT_HASH} ."
                withDockerRegistry([ credentialsId: "registry", url: "https://${env.REGISTRY_URL}" ]) {
                    sh "docker push ${env.REGISTRY_URL}/${env.NAMESPACE}/${env.PROJECT}:${GIT_COMMIT_HASH}"
                }
            }
        }

        stage('Clean') {
            steps {
               sh "docker rmi -f ${env.REGISTRY_URL}/${env.NAMESPACE}/${env.PROJECT}:${GIT_COMMIT_HASH}"
            }
        }

        stage('Confirm deploy to STAGE ?'){
            when {
                expression { DEPLOY_STAGE == "false" }
            }
            steps {
                script {
                    try {
                        timeout(time: 60, unit: 'SECONDS') {
                            input "Would you like to deploy elastalert version ${GIT_COMMIT_HASH} to STAGE ?"
                            DEPLOY_STAGE = true
                        }
                    }catch(err){}
                }
            }
        }

        stage('Deploy to stage') {
            when {
                expression { DEPLOY_STAGE != "false" }
            }
            steps {
                deploy serviceName: SERVICE_NAME, version: GIT_COMMIT_HASH, gitBranch: GIT_BRANCH, environment: 'stage'
            }
        }

    }
}