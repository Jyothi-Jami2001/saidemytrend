def registry = " https://trialrxbmgx.jfrog.io"
pipeline {
    agent any
    environment {
        PATH =  "/opt/maven/bin:$PATH"
    }

    stages {

        stage("build") {
            steps {
                 echo "-------build started-------"

                 sh 'mvn clean deploy -Dmaven.test.skip=true'

                 echo "-------build completed-------"
            }
        }

        stage("test") {
            steps {
                 echo "-------unit test  started-------"

                 sh 'mvn surefire-report:report'

                 echo "-------unit test  completed-------"
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


     stage("Quality Gate") {
        steps {
           script {
               timeout(time: 1, unit: 'HOURS') {
                    def qg = waitForQualityGate()
                     if (qg.status != 'OK') {
                        echo  "Warning: Quality Gate failed but continuing pipeline:  ${qg.status}"
                     }
               }
           } 
        }
      } 
    stage("Jar Publish") {
       steps {
          script {
              echo '<-----Jar Publish started----->'
              def server = Artifactory.newServer url: registry + "/artifactory", credentialsId: "artifact-cred"
              def properties = "buildId = ${env.BUILD_ID}, commitid = ${GIT_COMMIT}"
              def uploadSpec = """{
                  "files" : [
                   {
                     "pattern" : "jarstaging/(*)",
                     "target" : "jyo-libs-release-local/{1}",
                     "flat" : "false",
                     "props" : "${properties}",
                     "exclusions" : ["*.sha1", "*.md5"]
                   }
                ]
             }"""
             def buildInfo = server.upload(uploadSpec)
             buildInfo.env.collect()
             server.publishBuildInfo(buildInfo)
             echo '<-----Jar Publish ended------>'
          }
       }
    }
 }
 
}
