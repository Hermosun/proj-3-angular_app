// Jenkinsfile
pipeline {
    agent any
    
    parameters {
        string(name: 'VERSION', defaultValue: '1.0.1', description: 'Version to deploy')
        choice(name: 'DEPLOY_TYPE', choices: ['deploy', 'rollback'], description: 'Deploy new version or rollback to previous')
        string(name: 'ROLLBACK_VERSION', defaultValue: '', description: 'Version to rollback to (if DEPLOY_TYPE is rollback)')
    }
    
    stages {
        stage('Build Angular App') {
            when {
                expression { params.DEPLOY_TYPE == 'deploy' }
            }
            steps {
                sh 'npm install'
                sh 'npm run build --prod'
                sh "tar -czf angular-app-${params.VERSION}.tar.gz -C dist/angular-app ."
                archiveArtifacts artifacts: "angular-app-${params.VERSION}.tar.gz", fingerprint: true
            }
        }
        
        stage('Select Version for Rollback') {
            when {
                expression { params.DEPLOY_TYPE == 'rollback' }
            }
            steps {
                script {
                    def version = params.ROLLBACK_VERSION
                    if (!version) {
                        error "Rollback version ${params.ROLLBACK_VERSION} does not exist or was not found!"
                    }
                    // Copy artifact from previous build
                    copyArtifacts(projectName: env.JOB_NAME, filter: "angular-app-${params.ROLLBACK_VERSION}.tar.gz", selector: specific(params.ROLLBACK_VERSION))
                    env.DEPLOY_VERSION = version
                }
            }
        }
        
        stage('Deploy') {
            steps {
                script {
                    def versionToDeploy = params.DEPLOY_TYPE == 'deploy' ? params.VERSION : params.ROLLBACK_VERSION
                        env.VERSION_TO_DEPLOY = versionToDeploy
                    
                    // Copy the artifact to target server
                    sh "scp angular-app-${versionToDeploy}.tar.gz user@app_server:/tmp/"
                    
                    // Run Ansible playbook
                    sh """
                    cd /path/to/ansible
                    ansible-playbook playbooks/deploy.yml -i inventory -e "version=${versionToDeploy}" -e "artifact_path=/tmp/angular-app-${versionToDeploy}.tar.gz"
                    """
                }
            }
        }
    }
}