pipeline {
    agent {
        docker { 
            image 'node:8.11.3-jessie' 
            args '-u 0:0'
        }
    }
    stages {
        stage('Merge with prod org'){
            steps {
                sh 'npm install --global force-dev-tool --silent'
                withCredentials([usernamePassword(credentialsId: 'fulltet2Creds', passwordVariable: 'SF_PASSWORD', usernameVariable: 'SF_USERNAME')]) {
                    sh 'force-dev-tool remote add production ${SF_USERNAME} ${SF_PASSWORD} https://test.salesforce.com'
                }
                sh 'git checkout REALISE'
                withCredentials([usernamePassword(credentialsId: 'gitMellanox-salesforce', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
                    sh 'git config --global user.email ${GIT_USERNAME}'
                    sh 'git config --global user.name ${GIT_USERNAME}'
                }
                sh 'git checkout master'
                sh 'git checkout REALISE'
            }
        }
        stage('Running tests & dry-run deploy'){
            steps {
                sh 'git diff --no-renames --name-only master REALISE | tr \'\\n\' \' \''
                sh 'force-dev-tool changeset create deploy $(git diff --no-renames --name-only master REALISE | tr \'\\n\' \' \') -f'
                sh 'force-dev-tool deploy -c -d config/deployments/deploy production'
            }
        }
        stage('Running tests & dry-run deploy'){
             steps {
                sh 'force-dev-tool deploy -d config/deployments/deploy production'
             }
         }
    }
    post{
        always{
            sh 'git checkout master'
            sh 'git merge REALISE'
            sh 'rm -rf .git'
            sh 'rm -rf config'
            sh 'rm -rf src'
            sh 'rm -rf Jenkinsfile'
            sh 'rm -rf Jenkinsfile_scheduled'
            deleteDir()
        }
    }
}
