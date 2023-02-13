pipeline {
 	agent { node { label 'SILVER' } }
  options { 
        //only 1 build on the same time
        disableConcurrentBuilds(), 
        //log 10 build
        buildDiscarder(logRotator(numToKeepStr: '10')) 
    }
 
  tools {
 	    maven "MAVEN3"
 	    jdk "OracleJDK8"
 	}
 
  environment {
         registryCredential = 'ecr:us-east-1:awscreds'
         appRegistry = "750232146652.dkr.ecr.us-east-1.amazonaws.com/vprofileappimg"
         vprofileRegistry = "https://750232146652.dkr.ecr.us-east-1.amazonaws.com"
         cluster = "vprofile"
         service = "vprofileappsvs"
     }
  stages {
    stage('Fetch code'){
      steps {
         git branch: 'docker', url: 'https://github.com/svinsi/jenkins.git'
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

     stage('Build App Image') {
        steps {

          script {
                 dockerImage = docker.build( appRegistry + ":$BUILD_NUMBER", "./Docker-files/app/multistage/")
              }

      }

     }

     stage('Upload App Image') {
           steps{
             script {
               docker.withRegistry( vprofileRegistry, registryCredential ) {
                 dockerImage.push("$BUILD_NUMBER")
                 dockerImage.push('latest')
               }
             }
           }
      }

      stage('Deploy to ecs') {
           steps {
         withAWS(credentials: 'awscreds', region: 'us-east-1') {
           sh 'aws ecs update-service --cluster ${cluster} --service ${service} --force-new-deployment'
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
