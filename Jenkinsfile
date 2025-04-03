pipeline {
    agent any
    environment {
        PATH =  "/opt/maven/bin:$PATH"
    }

    stages {

        stage("build") {
            steps {
                 sh 'mvn clean deploy'
            }
        }
 
      stage("SonarQube analysis") {
          environment {
               scannerHome = tool 'Jyo-sonar-scanner'
          }

          steps {
               withSonarQubeEnv('Jyo-sonarqube-server') {

                    sh "${scannerHome}/bin/sonar-scanner"
                }
          }
       }
    } 
}
