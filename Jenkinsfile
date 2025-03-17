pipeline {
    agent any

    environment {
        TF_VAR_vsphere_user = credentials('vsphere_user')
        TF_VAR_vsphere_password = credentials('vsphere_user')
        TF_VAR_vsphere_server = 'your-vsphere-server'
        CSV_FILE = 'machines.csv'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git 'https://your-repo.git'
            }
        }

        stage('Generate Terraform Variables') {
            steps {
                sh 'python3 csv_to_tfvars.py'
            }
        }

        stage('Terraform Init') {
            steps {
                sh 'terraform init'
            }
        }

        stage('Terraform Apply') {
            steps {
                sh 'terraform apply -auto-approve -var-file=terraform.tfvars'
            }
        }

        stage('Join Windows Machines to Domain') {
            steps {
                script {
                    def windows_ips = sh(script: "grep -E 'windows' terraform.tfvars | awk -F'\"' '{print $4}'", returnStdout: true).trim().split("\n")
                    for (ip in windows_ips) {
                        sshagent(['windows-admin']) {
                            sh "ssh Administrator@${ip} 'powershell -ExecutionPolicy Bypass -File join_windows.ps1'"
                        }
                    }
                }
            }
        }

        stage('Join Linux Machines to Domain') {
            steps {
                script {
                    def linux_ips = sh(script: "grep -E 'linux' terraform.tfvars | awk -F'\"' '{print $4}'", returnStdout: true).trim().split("\n")
                    for (ip in linux_ips) {
                        sshagent(['linux-admin']) {
                            sh "ssh root@${ip} 'bash join_linux.sh'"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'terraform.tfvars'
            sh 'terraform destroy -auto-approve -var-file=terraform.tfvars' // Cleanup
        }
    }
}
