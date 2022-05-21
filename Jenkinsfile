@Library('jenkins-devops@dev')_ // load jenkins-devops as a library

properties([
  parameters([
    choice(      name: 'TYPE',              choices:     ['converge', 
                                                          'dismiss'],
                                            defaultValue: 'converge',       description: 'TYPE of run Deploy or Dismis packages'),
    string(      name: 'SERVICE',           defaultValue: 'endpoints',      description: 'Name of Service (also namespace too)'),
  ])
])

pipeline {
  agent { label 'multiwerf' }

  environment {
    WERF_TMP_DIR                = '/home/jenkins/tmp/'
    WERF_HOME                   = '/home/jenkins/tmp/.werf/'
    WERF_IMAGES_REPO_MODE       = "monorepo"
    
    YC_REGISTRY                 = 'cr.yandex/xxx'
    AWS_REGISTRY                = 'xxx.amazonaws.com'
    WERF_STAGES_STORAGE         = "cr.yandex/xxx/${params.SERVICE}/stages"

    YC_AUTH                     = credentials("ya_registry_key")
    KUBECONFIG                  = credentials('werf_kube_config')
    WERF_SECRET_KEY             = credentials("WERF_SECRET_KEY")
    
    AWS_ACCESS_KEY_ID           = credentials('aws_access_key_id')
    AWS_SECRET_ACCESS_KEY       = credentials('aws_secret_access_key')
  }

  stages {
    stage("Prerequisites") {
      steps {
        script {
          sh 'export'
        }
        script{
          functions.loginYCRegistry("${YC_AUTH}", "${YC_REGISTRY}")
        script {
          functions.loginAWSRegistry("${AWS_ACCESS_KEY_ID}", "${AWS_SECRET_ACCESS_KEY}", "${params.SERVICE}", "${AWS_REGISTRY}")
          }
        }
      }
    }
    stage("DEV") {
      when {
        anyOf { branch 'dev'; }
      }
      environment {   
        WERF_IMAGES_REPO      = "${YC_REGISTRY}/${params.SERVICE}"

        WERF_NAMESPACE        = "${params.SERVICE}"
        WERF_ENV              = "dev"
        WERF_KUBE_CONTEXT     = "dev"
      }
      steps {
        script {
          functions.runWerf("${params.TYPE}")
        }
      }
    }
    stage("PROD YC") {
      when {
        anyOf { branch 'master'; }
      }
      environment {
        WERF_IMAGES_REPO      = "${YC_REGISTRY}/${params.SERVICE}"

        WERF_NAMESPACE        = "${params.SERVICE}"
        WERF_ENV              = "production"
        WERF_KUBE_CONTEXT     = "production"
      }
      steps {
        script {
          functions.runWerf("${params.TYPE}")
        }
      }
    }
    stage("PROD AWS") {
      when {
        anyOf { branch 'master'; }
      }
      environment { 
        WERF_IMAGES_REPO      = "${AWS_REGISTRY}/${params.SERVICE}"

        WERF_NAMESPACE        = "${params.SERVICE}"
        WERF_ENV              = "aws-production"
        WERF_KUBE_CONTEXT     = "aws-production"
      }
      steps {
        script {
          functions.runWerf("${params.TYPE}")
        }
      }
    }
  }
}