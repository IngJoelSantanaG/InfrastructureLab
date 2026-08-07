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

     stage('Compile') {
          steps {
              sh 'mvn clean compile'
              
            }

         }

      stage('Run') {
           steps {
              sh 'java -cp target/classes HelloWorld'

             }

           }

         }

      post {
        always {
            echo 'Pipeline finished.'

          }

       success {
           echo 'Build SUCCESS'

          }

       failure {
           echo 'Build FAILED!'

         }

       }

    }

