pipeline {
   agent any

   options {
      timestamps()

       }
    
   parameters {
     string(
          name: 'APPLICATION_VERSION',
          defaultValue: '1.2',
          description: 'Application version to build and deploy'

          )

        }


      environment {

           KNOWN_GOOD_FILE = '/var/lib/jenkins/deployment-state/infrastructure-lab-known-good'



           }

     stages {
        
           stage('Environment') {
                steps {
                  sh 'echo "User: $(whoami)"'
                  sh 'echo "Workspace: $(pwd)"'
                  sh 'java --version'
                  sh 'mvn --version'

            }

         }

     stage('Build') {
          steps {
              sh "mvn versions:set -DnewVersion=${params.APPLICATION_VERSION}"
              sh 'mvn clean package'
              
            }

         }

      stage('Deploy') {
           steps {
              sh "cp target/infrastructure-lab-${params.APPLICATION_VERSION}.war /var/lib/tomcat10/webapps/"

             }

           }
      stage('Verify'){
          steps {
             script {
                def verificationResult = sh ( 

                  script: """
                   echo "Checking deployed application..."
                
                  for i in \$(seq 1 12); do
                       echo "Verification attempt \$i..."

                 if  curl -fsS http://localhost:8080/infrastructure-lab-${params.APPLICATION_VERSION}/ \\
                   | grep -q "Infrastructure Lab v${params.APPLICATION_VERSION}"; then

                       echo "Application verification successful!"
                       exit 0
                   fi 
                 
                  echo "Application not ready yet,waiting 5 seconds..."
                  sleep 5

                done

                  echo "Application verification FAILED!"
                  exit 1

               """,
               returnStatus: true

             )

             if (verificationResult !=0){
                   echo "Deployment verification failed."
                   echo "Rollback will be attempted."
                   env.DEPLOYMENT_FAILED = 'true'
                 }
               }
             }
           }

      stage ('Update Known-Good Version') {
         when {
          expression {
               env.DEPLOYMENT_FAILED != 'true'
            
             }
           }
          
        steps {

          sh """
              echo "Updating known-good version to ${params.APPLICATION_VERSION}..."
              echo "${params.APPLICATION_VERSION}" > ${env.KNOWN_GOOD_FILE}

             echo "Known-good version is now:"
             cat ${env.KNOWN_GOOD_FILE}
 

           """
        }
      }
         
      stage ('Rollback'){
           when {
                  environment name: 'DEPLOYMENT_FAILED', value: 'true'
               }

             steps {
                  
                 script {

                    env.ACTUAL_KNOWN_GOOD = sh (
                         script: "cat ${env.KNOWN_GOOD_FILE}",
                         returnStdout: true
                       ).trim()

                    echo "Recovered known-good version: ${env.ACTUAL_KNOWN_GOOD}"


                     }


                 sh """
                     echo "========================================"
                     echo "ROLLBACK STARTED"
                     echo "========================================"

                     echo "Removing failed ${params.APPLICATION_VERSION} WAR..."
                     rm -f /var/lib/tomcat10/webapps/infrastructure-lab-${params.APPLICATION_VERSION}.war

                     echo "Known-good version: ${env.ACTUAL_KNOWN_GOOD}"
                     echo "Waiting for application recovery..."

                   for i in \$(seq 1 12); do
                          echo "Rollback verification attempt \$i..."

                      if curl -fsS http://localhost:8080/infrastructure-lab-${env.ACTUAL_KNOWN_GOOD}/ \\
                         | grep -q "Infrastructure Lab v${env.ACTUAL_KNOWN_GOOD}"; then
                           
                            echo "================================="
                            echo "ROLLBACK SUCCESSFUL"
                            echo "Infrastructure Lab ${env.ACTUAL_KNOWN_GOOD} is healthy"
                            echo "================================="

                           exit 0

                  fi

                 echo "Known-good version ${env.ACTUAL_KNOWN_GOOD} not ready yet, Waiting 5 seconds..."
                 sleep 5
               done

                   echo "======================================="
                   echo "ROLLBACK FAILED"
                   echo "======================================="

                  exit 1

              """

          }         

         }

       }

        
      post {
        always {
            echo 'Pipeline finished.'

          }

       success {
           echo 'Build or deployment SUCCESS'

          }

       failure {
           echo 'Build or deployment FAILED'
       
       }
     
     }
  }
