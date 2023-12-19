/* import shared library */
@Library('jenkins-shared-library')_


pipeline {
    agent any
    stages {
      stage('Setup parameters') {
          
            steps {
                script {
                    properties([
                        parameters([    
                        
                        choice(
                            choices: ['dev','main'], 
                            name: 'Environment'   
                                ),
                      ])
                    ])
                }
            }
        }
       
        stage('Prepare ansible environment') {
           
            environment {
                VAULTKEY = credentials('vaultkey')
                DEVOPSKEY = credentials('devopskey')
            }
            steps {
                sh 'echo \$VAULTKEY > vault.key'
                sh 'cp \$DEVOPSKEY id_rsa'
                sh 'chmod 700 id_rsa'
            }
        }
        stage('Test and deploy the application in preproduction') {
            
            stages {
               stage("Install ansible role dependencies") {
           
                   steps {
                       sh 'ansible-galaxy install  -r roles/requirements.yml'
                   }
               }
               /*stage("Ping targeted hosts") {
                   steps {
                       sh 'ansible all -m ping -i hosts --private-key=id_rsa'
                   }
               }*/
               /*stage("Verify ansible playbook syntax") {
                   steps {
                       sh 'sudo pip install ansible-lint'
                       sh 'ansible-lint -x 306 install_fake-backend.yml'
                       sh 'echo "${GIT_BRANCH}"'
                   }
               }*/

             
               stage("Build docker images on build host") {
                   when{ 
                      expression {
                        env.Environment == 'main' }
                    }
                   steps {
                       sh 'ansible-playbook  -i hosts --vault-password-file vault.key --private-key id_rsa --tags "build" --limit localhost install_fake-backend.yml'
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
