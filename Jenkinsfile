#!/usr/bin/env groovy

pipeline {
  
  environment {
    STACK_NAME = "demo1"
    AWS_REGION = "us-east-1"
  }

  agent any

  stages {
    stage('Inspect Template') {
      steps {
        build job: 'cfn-template-inspector', parameters: [
          [$class: 'StringParameterValue', name: 'workspaceDir', value: "${env.WORKSPACE}"],
          [$class: 'StringParameterValue', name: 'templateFilename', value: 'stack.yaml'],
          [$class: 'BooleanParameterValue', name: 'runInspector', value: Boolean.valueOf("false")]
        ]
      }
    }
    stage('Create/Update Stack') {
      steps {
        script {
          def stackExists = false

          try {
            sh "aws cloudformation --region ${env.AWS_REGION} describe-stacks --stack-name ${env.STACK_NAME} >&/dev/null"
            stackExists = true
            echo "Stack ${env.STACK_NAME} already exists"
          } catch(err) {
            echo "Stack ${env.STACK_NAME} does not exist"
            // Do nothing
          }

          if(stackExists) {
            echo "Updating CloudFormation stack ${env.STACK_NAME}"
            sh "aws cloudformation --region ${env.AWS_REGION} update-stack --stack-name ${env.STACK_NAME} --template-body file://`pwd`/stack.yaml"
            sh "aws cloudformation --region ${env.AWS_REGION} wait stack-update-complete --stack-name ${env.STACK_NAME}"
          } else {
            echo "Creating CloudFormation stack ${env.STACK_NAME}"
            sh "aws cloudformation --region ${env.AWS_REGION} create-stack --stack-name ${env.STACK_NAME} --template-body file://`pwd`/stack.yaml"
            sh "aws cloudformation --region ${env.AWS_REGION} wait stack-create-complete --stack-name ${env.STACK_NAME}"
          }
        }
      }
    }
  }
} 
