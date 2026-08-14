pipeline {
   agent any

   options {
      timestamps()

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
              sh '''
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
            
        sh '''
            echo "========================================"
            echo "ROLLBACK STARTED"
            echo "========================================"

            echo "Removing failed 1.2 war..."

            rm -f /var/lib/tomcat10/webapps/infrastructure-lab-1.2.war
            
            echo "Deploying known-good 1.1..."
           cp /var/lib/tomcat10/webapps/infrastructure-lab-1.1.war \
              /var/lib/tomcat10/webapps
            
            echo "Waiting for Tomcat to deploy 1.1..."

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
         }

       }

    }

