pipeline {
   agent any

     stages {
           stage('Display Information') {
                steps {
                  sh 'pwd'
                  sh 'ls -la'

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

      }
