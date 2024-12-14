pipeline {
    agent {
        label 'AGENT-1'
    }

    options {
        timeout(time: 10, unit: 'SECONDS')
        disableConcurrentBuilds()
        // retry(1)
    }
    environment {
        DEBUG = 'true'
        appVersion = '' // this will become global, we can use across pipeline
        region = 'us-east-1'
        account-id = '337909750491'
        project= 'expense'
        environment = 'dev'
        component= 'frontend'
    }
      
      stages{
        stage('Read the version') {
            steps {
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "App version: ${appVersion}"
                }
            }
        }

    

         stage('Docker build') {
            
            steps {
                withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                    sh """
                    aws ecr get-login-password --region ${region} | docker login --username AWS --password-stdin ${account_id}.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion} .

                    docker images

                    docker push ${account_id}.dkr.ecr.us-east-1.amazonaws.com/${project}/${environment}/${component}:${appVersion}
                    """
                }
            }

         }

         stage('deploy') {
            steps{
                withAWS(region: 'us-east-1', credentials: 'aws-creds'){
                  sh """
                    aws eks update-kubeconfig --region ${region} --name ${project}-${environmet}
                    cd helm
                    sed -i 's/IMAGE_VERSION/${appVersion}/g' values-${environment}.yaml
                    helm upgrade --install  ${component} -n ${project} -f values-${environment}.yaml .

                  """

                }

            }
         }

        }



         // stage('Approval'){
        //     input {
        //         message "Should we continue?"
        //         ok "Yes, we should."
        //         submitter "alice,bob"
        //         parameters {
        //             string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
        //         }
        //     }
        //     steps {
        //         echo "Hello, ${PERSON}, nice to meet you."
        //     }
        // }
    }

    post {
        always {
            echo " this section always runs"
            // deleteDir()
        }

        success {
            echo " this section runs when pipeline is success"
        }
        failure {
            echo " this section runs when pipeline is failed"
        }
    }

}
}