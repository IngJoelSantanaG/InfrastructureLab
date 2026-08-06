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
                  sh 'git --version'

            }

         }

     stage('Compile') {
          steps {
              sh 'javac HelloWorld.java'
              
            }

         }

      stage('Run') {
           steps {
              sh 'java HelloWorld'

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

