pipeline {
    agent {
        label '007'
    }
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "auto_maven"
        terraform 'Terraform'
    }
    environment {
        IMAGE = sh script: 'mvn help:evaluate -Dexpression=project.artifactId -q -DforceStdout', returnStdout: true

        VERSION = sh script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true

    }

    stages {
        stage('Clear'){
            steps{
                //clear 
                sh 'docker rm -f pandaapp || true'
            }
        }
        stage('Build') {
            steps {

                // Run Maven on a Unix agent.
                sh "mvn -Dmaven.test.failure.ignore=true clean install"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }
        stage ('Docker image'){

            steps{
                sh "mvn package -Pdocker"
            }
        }
        stage('Run Docker app') {
            steps {
                sh "docker run -d -p 8080:8080 --name pandaapp ${IMAGE}:${VERSION}"
            }
        } 
        stage('Deploy'){
            steps{
                configFileProvider([configFile(fileId: '636b1cbf-d42c-411e-a6a4-49912261e4b4', variable: 'MAVEN_GLOBAL_SETTINGS')]) {
                    sh 'mvn -gs $MAVEN_GLOBAL_SETTINGS deploy -Dmaven.test.skip=true'
                }
            }
        }

        stage('Run terraform') {
            steps {
                dir('infrastructure/terraform') { 
                    withCredentials([file(credentialsId: 'panda', variable: 'terraformpanda')]) {
                        sh "cp \$terraformpanda ../panda.pem"
                    }
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS']]) {
                        sh 'terraform init && terraform apply -auto-approve -var-file panda.tfvars'
                    }
                } 
            }
        }
        stage('Copy Ansible role') {
            steps {
                sh 'sleep 180'
                sh 'cp -r infrastructure/ansible/panda/ /etc/ansible/roles/'
            }
        }

        stage('Run Ansible') {
            steps {
                dir('infrastructure/ansible') { 
                    sh 'chmod 600 ../panda.pem'
                    sh 'ansible-playbook -i ./inventory playbook.yml -e ansible_python_interpreter=/usr/bin/python3'
                } 
            }
        }

        stage('Remove environment') {
            steps {
                input 'Remove environment'
                dir('infrastructure/terraform') { 
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS']]) {
                        sh 'terraform destroy -auto-approve -var-file panda.tfvars'
                    }
                }
            }
        }
    }    
    post {
        success {
            sh 'docker stop pandaapp'
            deleteDir()
        }
        failure {
            dir('infrastructure/terraform') { 
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'AWS']]) {
                    sh 'terraform destroy -auto-approve -var-file panda.tfvars'
                }
            }
            sh 'docker stop pandaapp'
            deleteDir()
        }
    }
   

    
}