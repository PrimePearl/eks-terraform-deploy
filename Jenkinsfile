pipeline {
  agent { node { label "terraform-node" } }

  parameters {
    choice(name: 'deploy_choice', choices: ['apply', 'destroy'], description: 'The deployment type')
  }

  environment {
    EMAIL_TO = 'your_email@domain.com' // Consider using an environment variable here
    AWS_REGION = 'us-east-1'
  }

  stages {
    stage('1. Terraform Init') {
      steps {
        echo 'Terraform init phase'
        sh 'terraform init'
      }
    }

    stage('2. Terraform Plan') {
      steps {
        echo 'Terraform plan phase'
        sh 'terraform plan'
      }
    }

    stage('3. Manual Approval') {
      steps {
        script {
          def userInput = input message: "Should we proceed?", ok: "Yes, we should.", parameters: [
            choice(name: 'Manual_Approval', choices: ['Approve', 'Reject'], description: 'Approve or Reject the deployment')
          ]
          echo "Deployment ${userInput}"
        }
      }
    }

    stage('4. Terraform Deploy') {
      when {
        expression {
          params.deploy_choice == 'apply' || params.deploy_choice == 'destroy'
        }
      }
      steps {
        echo "Terraform ${params.deploy_choice} phase"
        sh """
          terraform ${params.deploy_choice} ${params.TARGET_MODULES ? '-target=' + params.TARGET_MODULES : ''} --auto-approve=false  # Avoid auto-approve in production
          aws eks --region ${env.AWS_REGION} update-kubeconfig --name dominion-cluster && export KUBE_CONFIG_PATH=~/.kube/config
          terraform ${params.deploy_choice} --auto-approve=false  # Avoid auto-approve in production
        """
      }
    }

    stage('5. Email Notification') {
      steps {
        mail bcc: '${env.EMAIL_TO}',
              body: '''Terraform deployment is completed.
                Let me know if the changes look okay.
                Thanks,
                Dominion System Technologies,
                +233 (050) 193-4917''',
              cc: '${env.EMAIL_TO}',
              from: '',
              replyTo: '',
              subject: 'Terraform Infra deployment completed!!!',
              to: '${env.EMAIL_TO}'
      }
    }
  }
}
