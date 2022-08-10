pipeline {
  agent {
    node {
      label 'build-terraform-aws'
    }
  }
  options {
    ansiColor('xterm')
    disableConcurrentBuilds()
  }
  parameters {
    choice(choices: ['create','destroy'], description: 'TF action', name: 'ACTION')
    string(name: 'TARGET', defaultValue: '', description: 'Target a specific resource instead of everything: https://www.terraform.io/docs/internals/resource-addressing.html')
    booleanParam(defaultValue: false, description: 'Apply TF action after plan', name: 'DEPLOY')
    string(name: 'ENV_NAME', defaultValue: 'test', description: 'Terraform workspace to use for deployment.')
  }

  environment {
    TERRAFORM="/home/jenkins/.terraform.versions/terraform_0.15.4"
  }

  stages {
    stage('Init') {
      steps {
        script {
          if ( params.TARGET != "" )
          {
            env.TARGET = "-target=${params.TARGET}"
          }
        }
        sh 'cd $WORKSPACE/fbs_tf/envs/${ENV_NAME}; $TERRAFORM init'
        sh 'cd $WORKSPACE/fbs_tf/envs/${ENV_NAME}; terraform workspace select ${ENV_NAME} || terraform workspace new ${ENV_NAME} && terraform workspace select ${ENV_NAME}; $TERRAFORM init; $TERRAFORM validate'
      }
    }
    stage('Plan') {
      steps {
        script {
          if ( params.ACTION == "destroy" )
          {
            sh 'cd $WORKSPACE/fbs_tf/envs/${ENV_NAME}; terraform workspace select ${ENV_NAME} || terraform workspace new ${ENV_NAME} && terraform workspace select ${ENV_NAME}; $TERRAFORM plan -destroy $TARGET'
          }
          else
          {
            sh 'cd $WORKSPACE/fbs_tf/envs/${ENV_NAME}; terraform workspace select ${ENV_NAME} || terraform workspace new ${ENV_NAME} && terraform workspace select ${ENV_NAME}; $TERRAFORM init; $TERRAFORM plan $TARGET'
          }
        }
      }
    }    
    stage('Create') {
      when {
        expression {
            params.DEPLOY == true && params.ACTION == "create"
        }
      }
      steps {
        echo 'Deploying to the ${ENV_NAME} environment'
        sh 'cd $WORKSPACE/fbs_tf/envs/${ENV_NAME}; terraform workspace select ${ENV_NAME} || terraform workspace new ${ENV_NAME} && terraform workspace select ${ENV_NAME}; $TERRAFORM apply -auto-approve -parallelism=20 $TARGET'
      }
    }
    stage('Destroy') {
      when {
        expression {
          params.DEPLOY == true && params.ACTION == "destroy"
        }
      }
      steps {
        echo 'Deploying to the ${env.BRANCH_NAME} environment'
        sh 'cd $WORKSPACE/fbs_tf/envs/${ENV_NAME}; terraform workspace select ${ENV_NAME} || terraform workspace new ${ENV_NAME} && terraform workspace select ${ENV_NAME}; $TERRAFORM destroy -auto-approve -parallelism=20 $TARGET'
      }
    }
  }  // end stages
}  // end pipeline
