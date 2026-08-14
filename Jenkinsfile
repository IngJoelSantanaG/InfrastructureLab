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
           echo 'Build or deployment FAILED!'

         }

       }

    }

