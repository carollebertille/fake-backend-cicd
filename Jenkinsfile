/* import shared library */
@Library('jenkins-shared-library')_


pipeline {
    agent none
    stages {
       
        stage('Prepare ansible environment') {
            agent any
            environment {
                VAULTKEY = credentials('vaultkey')
                DEVOPSKEY = credentials('devopskey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
                sh 'cp \$DEVOPSKEY id_rsa'
                sh 'chmod 600 id_rsa'
            }
        }
        stage('Test and deploy the application in preproduction') {
             agent none
            stages {
               stage("Install ansible role dependencies") {
            agent any
                   steps {
                       sh 'ansible-galaxy install  -r roles/requirements.yml'
                   }
               }
               stage("Ping targeted hosts") {
                   steps {
                       sh 'ansible all -m ping -i hosts --private-key id_rsa'
                   }
               }
               stage("Verify ansible playbook syntax") {
                   steps {
                       sh 'ansible-lint -x 306 install_fake-backend.yml'
                       sh 'echo "${GIT_BRANCH}"'
                   }
               }

              
               stage("Build docker images on build host") {
                   when {
                      expression { GIT_BRANCH == 'origin/dev' }
                   }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "build" --limit build install_fake-backend.yml'
                   }
               }



               }
            }
         }
    
    
    post {
    always {
       script {
         /* Use slackNotifier.groovy from shared library and provide current build result as parameter */
         clean
         slackNotifier currentBuild.result
     }
    }
    }
}
