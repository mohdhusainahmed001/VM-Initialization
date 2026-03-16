pipeline {
    agent {
        label 'builtin'
    }

    parameters {
        choice(name: 'project',
            choices: ['engage', 'omni-api', 'warcs', 'odp', 'odp-lakehouse'],
            description: 'Select the environment to Initialize')
    }

    stages {
        stage("Dry run ansible playbook") {
            steps {
                ansiblePlaybook(
                    playbook: "playbooks/${params.project}.yml",
                    inventory: "inventory/hosts",
                    // Add this line
                    vaultCredentialsId: 'ansible-vault-pass',
                    extras: '--check'
                )
            }
         }

        stage("Ask for approval") {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input(
                        id: 'proceed',
                        message: 'Are you sure you want to deploy?', 
                        ok: 'Continue Deployment'
                    )
                }
            }
        }

        stage("Run ansible playbook") {
            steps {
                ansiblePlaybook(
                    playbook: "playbooks/${params.project}.yml",
                    inventory: "inventory/hosts",
                    // Add this line here as well
                    vaultCredentialsId: 'ansible-vault-pass'
                )
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}