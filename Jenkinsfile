pipeline {
    agent any
    environment {
     TF_HOME = tool('tf-1.7')
     ENV_auto_tfvars = credentials('digitalocean_terraform_variables')
    }

    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/pg_local_backend']], extensions: [], userRemoteConfigs: [[credentialsId: 'git_priv', url: 'https://bitbucket.org/szkoleniacloud/cw28-tf-digitalocean.git']])
            }
        }
        stage('Terraform INIT') {
            steps {
                script {
                    sh'''
                    set -e
                    #$TF_HOME/terraform -version
                    #$TF_HOME/terraform init -backend-config="conn_str=postgresql://postgres:terraform@localhost:5432/terraform?sslmode=disable"
                    export PATH=$PATH:/var/lib/jenkins/tools/org.jenkinsci.plugins.terraform.TerraformInstallation/tf-1.7
                    docker run -t --user $(id -u):$(id -g) -v $(pwd):/app -w /app hashicorp/terraform:1.7.5 init -backend-config="conn_str=postgresql://postgres:terraform@10.248.160.11:5432/terraform?sslmode=disable"
                    '''
                }
            }
        }
        stage('Terraform VALIDATE') {
            steps {
                sh'''
                set -e
                #$TF_HOME/terraform validate
                #terraform init -backend-config="conn_str=postgresql://postgres:terraform@localhost:5432/terraform?sslmode=disable"
                docker run -t --user $(id -u):$(id -g) -v $(pwd):/app -w /app hashicorp/terraform:1.7.5 validate
                #terraform validate
                '''
            }
        }
        stage('Terraform PLAN') {
            steps {
                withCredentials([file(credentialsId: 'digitalocean_terraform_variables', variable: 'auto_tfvars')]) {
                    ansiColor('xterm') {
                        sh'''
                        set -e
                        echo "auto_tfvars: $auto_tfvars"
                        echo "ENV_auto_tfvars: $ENV_auto_tfvars"
                        cat $auto_tfvars
                        cat $ENV_auto_tfvars
                        #$TF_HOME/terraform plan -var-file=${auto_tfvars} -input=false -out=tfplan
                        docker run -t --user $(id -u):$(id -g) -v $(pwd):/app -v $ENV_auto_tfvars:/secret/.auto.tfvars:ro -w /app hashicorp/terraform:1.7.5 plan -input=false -out=tfplan -var-file=/secret/.auto.tfvars
                        '''
                    }
                }
            }
        }
        stage('Terraform APPLY') {
            steps {
                ansiColor('xterm') {
                    sh'''
                    set -e
                    #$TF_HOME/terraform apply -input=false tfplan
                    docker run -t --user $(id -u):$(id -g) -v $(pwd):/app -w /app hashicorp/terraform:1.7.5 apply tfplan
                    '''
                }
            }
        }
    }
}
