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
              sh 'cp target/infrastructure-lab-1.0.war var/lib/tomcat10/webapps/'

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

