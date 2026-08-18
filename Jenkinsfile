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
              sh 'cp target/infrastructure-lab-1.2.war /var/lib/tomcat10/webapps/'

             }

           }
      stage('Verify'){
          steps {
             script {
                def verificationResult = sh ( 

                  script: '''
                   echo "Checking deployed application..."
                
                  for i in $(seq 1 12); do
                       echo "Verification attempt $i..."

                 if  curl -fsS http://localhost:8080/infrastructure-lab-1.2/ \
                   | grep -q "Infrastructure Lab v1.2"; then

                       echo "Application verification successful!"
                       exit 0
                   fi 
                 
                  echo "Application not ready yet,waiting 5 seconds..."
                  sleep 5

                done

                  echo "Application verification FAILED!"
                  exit 1

                ''',
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
         
      stage ('Rollback'){
           when {
                  environment name: 'DEPLOYMENT_FAILED', value: 'true'
               }

             steps {
                 sh '''
                     echo "========================================"
                     echo "ROLLBACK STARTED"
                     echo "========================================"

                     echo "Removing failed 1.2 WAR..."
                     rm -f /var/lib/tomcat10/webapps/infrastructure-lab-1.2.war

                     echo "Known-good 1.1 WAR is already deployed."
                     echo "Waiting for application recovery..."

                   for i in $(seq 1 12); do
                          echo "Rollback verification attepmt $i..."

                      if curl -fsS http://localhost:8080/infrastructure-lab-1.1/ \
                         | grep -q "Infrastructure Lab v1.1"; then
                           
                            echo "================================="
                            echo "ROLLBACK SUCCESSFUL"
                            echo "Infrastructure Lab 1.1 is healthy"
                            echo "================================="

                           exit 0

                  fi

                 echo "1.1 not ready yet, Waiting 5 seconds..."
                 sleep 5
               done

                   echo "======================================="
                   echo "ROLLBACK FAILED"
                   echo "======================================="

                  exit 1

              '''

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
           echo 'Build or deployment FAILED - starting rollback'
          

       script {

        try {

        sh '''
            echo "========================================"
            echo "ROLLBACK STARTED"
            echo "========================================"

            echo "Removing failed 1.2 war..."

            rm -f /var/lib/tomcat10/webapps/infrastructure-lab-1.2.war
            
            echo "Known-good 1.1 WAR is already deployed."
            echo "Waiting for application recovery..."

            for i in $(seq 1 12); do
             echo "Rollback verification attempt $i..."
             
            if curl -fsS http://localhost:8080/infrastructure-lab-1.1/ \
               | grep -q "Infrastructure Lab v1.1"; then

             echo "======================================="
             echo "ROLLBACK SUCCESSFUL"
             echo "Infrastructure Lab 1.1 is healthy"
             echo "======================================="

             exit 0

          fi

             echo "1.1 not ready yet. waiting 5 seconds"
             sleep 5

         done

             echo "======================================="
             echo "ROLLBACK FAILED"
             echo "======================================="
             exit 1    
                        

           
            '''
            currentBuild.result = 'UNSTABLE'
            echo 'Deployment failed, but automatic rollback succeeded.'
            echo 'System recovered successfully to Infrastructure Lab 1.1.'

         } catch (Exception e){
            echo 'AUTOMATIC ROLLBACK FAILED!'
            currentBuild.result = 'FAILURE'
            throw e           
          
          }

         }

       }
     
     }
  }
