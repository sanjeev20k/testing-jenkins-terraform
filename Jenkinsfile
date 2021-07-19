
pipeline {

    parameters {
        string(name: 'environment', defaultValue: 'terraform', description: 'Workspace/environment file to use for deployment')
        booleanParam(name: 'autoApprove', defaultValue: false, description: 'Automatically run apply after generating plan?')

    }


     environment {
        AWS_ACCESS_KEY_ID     = credentials('AWS_ACCESS_KEY_ID')
        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
    }

   agent  any
    stages {
	GITHUB_PROJECT = “https://github.com/sanjeev20k/testing-jenkins-terraform.git”
	GITHUB_BRANCH = ‘${env.BRANCH_NAME}’
	node{
	    stage (“Listing Branches”) {
		echo “Initializing workflow”
		echo GITHUB_PROJECT
		git url: GITHUB_PROJECT, credentialsId: GITHUB_CREDENTIALS_ID
		sh ‘git branch -r | awk \'{print $1}\’ ORS=\’\\n\’ >branches.txt’
		sh ”’cut -d ‘/’ -f 2 branches.txt > branch.txt”’

	    }
	    stage(‘get build branch Parameter User Input’) {

		liste = readFile ‘branch.txt’
		echo “please click on the link here to chose the branch to build”
		env.BRANCH_SCOPE = input message: ‘Please choose the branch to build ‘, ok: ‘Validate!’,
		parameters: [choice(name: ‘BRANCH_NAME’, choices: “${liste}”, description: ‘Branch to build?’)]
	    }
	    stage(‘Checkout external proj’) {
		echo “${env.BRANCH_SCOPE}”
		git branch: “${env.BRANCH_SCOPE}”,
		url: ‘https://github.com/sanjeev20k/testing-jenkins-terraform.git’

		sh “ls -lat”
	    }
	
        stage('checkout') {
            steps {
                 script{
                        dir("terraform")
                        {
                            git "https://github.com/sanjeev20k/testing-jenkins-terraform.git"
                        }
                    }
                }
            }

        stage('Plan') {
            steps {
                sh 'terraform init -input=false'
                sh 'terraform workspace new ${environment}'
                sh 'terraform workspace select ${environment}'
                sh "terraform plan -input=false -out tfplan "
                sh 'terraform show -no-color tfplan > tfplan.txt'
            }
        }
        stage('Approval') {
           when {
               not {
                   equals expected: true, actual: params.autoApprove
               }
           }

           steps {
               script {
                    def plan = readFile 'tfplan.txt'
                    input message: "Do you want to apply the plan?",
                    parameters: [text(name: 'Plan', description: 'Please review the plan', defaultValue: plan)]
               }
           }
       }

        stage('Apply') {
            steps {
                sh "terraform apply -input=false tfplan"
            }
        }
    }
}
