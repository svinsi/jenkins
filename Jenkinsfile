pipeline {
 	agent { node { label 'SILVER' } }
  options { 
        //only 1 build on the same time
        disableConcurrentBuilds() 
        //log 10 build
        buildDiscarder(logRotator(numToKeepStr: '10')) 
    }
 
  tools {
 	    maven "MAVEN3"
 	    jdk "OracleJDK8"
 	}
 
  environment {
         registryCredential = 'ecr:us-east-1:awscreds'
         webImg = "vprofileweb"
         appImg = "vprofileappimg"
         dbImg  = "vprofiledb"
         ecrReg = "750232146652.dkr.ecr.us-east-1.amazonaws.com/"
     }
  stages {
    stage('Fetch code'){
      steps {
         git branch: 'main', url: 'https://github.com/svinsi/vprofileproject.git'
       }
     }


     stage('Test'){
       steps {
         sh 'mvn test'
       }
     }

     stage ('CODE ANALYSIS WITH CHECKSTYLE'){
             steps {
                 sh 'mvn checkstyle:checkstyle'
             }
             post {
                 success {
                     echo 'Generated Analysis Result'
                 }
             }
         }

         stage('build && SonarQube analysis') {
             environment {
              scannerHome = tool 'sonar4.7'
           }
             steps {
                 withSonarQubeEnv('sonar') {
                  sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                 }
             }
         }

         stage("Quality Gate") {
             steps {
                 timeout(time: 1, unit: 'HOURS') {
                     // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                     // true = set pipeline to UNSTABLE, false = don't
                     waitForQualityGate abortPipeline: true
                 }
             }
         }

     stage('Build Web Image') {
        steps {

          script {
                 dockerImage = docker.build( ecrReg +  webImg + ":$BUILD_NUMBER", "./Docker-files/web/")
              }

      }

     }

     stage('Upload Web Image') {
           steps{
             script {

               docker.withRegistry( "https://" + ecrReg, registryCredential ) {
                 dockerImage.push("$BUILD_NUMBER")
                 dockerImage.push('latest')
               }

             }
           }
      }

     stage('Build App Image') {
        steps {

          script {
                 dockerImage = docker.build( ecrReg +  appImg + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
              }

      }

     }

     stage('Upload App Image') {
           steps{
             script {
               docker.withRegistry( "https://" + ecrReg, registryCredential ) {
                 dockerImage.push("$BUILD_NUMBER")
                 dockerImage.push('latest')
               }
             }
           }
      }

        stage('Build db Image') {
        steps {

          script {
                 dockerImage = docker.build( ecrReg +  appImg + ":$BUILD_NUMBER", "./Docker-files/db/")
              }

      }

     }

     stage('Upload db Image') {
           steps{
             script {
               docker.withRegistry( "https://" + ecrReg, registryCredential ) {
                 dockerImage.push("$BUILD_NUMBER")
                 dockerImage.push('latest')
               }
             }
           }
      }

      stage('Run Docker-compose'){
            steps {
              script {
              withCredentials([sshUserPrivateKey(credentialsId: 'sshdockervm', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                    sh """
                    rsync -av --delete -e "ssh -o StrictHostKeyChecking=no -i ${identity}" $WORKSPACE/compose/docker-compose.yml  ${userName}@172.31.19.115:/tmp/
                    """
                docker.withRegistry( "https://" + ecrReg, registryCredential ) {
                sh """ 
                ssh -o StrictHostKeyChecking=no -i ${identity} ${userName}@172.31.19.115 '
                docker pull ${ ecrReg + webImg + ':latest' }
                docker pull ${ ecrReg + appImg + ':latest' }
                docker pull ${ ecrReg + dbImg + ':latest' }
                '
                """
           sh 'ssh -o StrictHostKeyChecking=no -i ${identity} ${userName}@172.31.19.115 "docker compose -f /tmp/docker-compose.yml up -d"'
           sh 'ssh -o StrictHostKeyChecking=no -i ${identity} ${userName}@172.31.19.115 "docker ps -a"'
              }
    }
       }
     }
    }

   }
 post { 
        // Clean after build 
        always { 
            cleanWs() 
        } 
    }
}
