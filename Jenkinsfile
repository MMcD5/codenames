pipeline {
  agent any
  options {
    disableConcurrentBuilds()
  }  
  environment {
    COMMIT_HASH = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
    MAIN_BRANCHES = "master,main"
    MERGED_BRANCH = get_merged_branch()
    F8_TAG = "$BRANCH_NAME-$BUILD_ID-$COMMIT_HASH"
    F8_ENVIRONMENT = "$BRANCH_NAME"
    F8_ENV_TYPE = "dev"
    F8_LOG_LINKS = "true"

    BUILD_CMD = "f8 build --push"
    DEPLOY_CMD = "f8 deploy"
    TEST_CMD = "f8 test"
  }

  stages {
    if (env.MAIN_BRANCHES.split(',').contains(env.BRANCH_NAME) && env.MERGED_BRANCH) {
      stage('Delete Env ' + merged_branch) {
        sh 'f8 delete --env ' + env.MERGED_BRANCH
      }
    }
    stage('Build') {
      steps {
        script {
          if (env.REG_USER != "") { 
            docker.withRegistry(env.REG_AUTH_URL, env.REG_CREDS_ID) {
              sh env.BUILD_CMD
            }
          } else {
            sh env.BUILD_CMD
          }
        }
      }
    }
    stage('Deploy') {
      steps {
        sh env.DEPLOY_CMD
      }
    }
    stage('Test') {
      steps {
        sh env.TEST_CMD
      }
    }
  }
  if (env.SLACK_ENABLED == 'true') {
    post {
      success {
        slackNotify('SUCCESS')
      }
      failure {
        slackNotify('FAILURE')
      }
    }
  }
}
