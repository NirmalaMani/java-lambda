pipeline {
    agent any

     options {
        //Disable concurrentbuilds for the same job
        disableConcurrentBuilds()
        // Colorize the console log
        ansiColor("xterm")          
        // Add timestamps to console log
        timestamps()
        
    }

    environment {
        AWS_ACCESS_KEY = credentials('aws_access_key')
        AWS_SECRET_KEY = credentials('aws_secret_key')
        ARTIFACTID = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
    }

    stages {
        stage('Build Lambda') {
            agent any
            steps {
                echo 'Build'
                sh 'mvn clean install -Dmaven.test.skip=true'             
            }
        }

        stage('Test') {
            agent any
            steps {
                echo 'Test'
                // sh 'mvn test'
            }
        }

        stage('Push to artifactory') {
            agent none
            steps {
                echo 'Push to artifactory'
            }
        }

        stage('Deploy to QA') {
            agent any
            steps {
                script {
                    echo 'Deploy to QA'
                    echo "ARTIFACTID: ${ARTIFACTID}"
                    echo "VERSION: ${VERSION}"
                    JARNAME = ARTIFACTID+'-'+VERSION+'.jar'
                    echo "JARNAME: ${JARNAME}"
                    sh 'pwd'
                    // sh "zip ${ARTIFACTID}-${VERSION}.zip 'target/${JARNAME}'"            

                    sh 'aws configure set aws_access_key_id $AWS_ACCESS_KEY'
                    sh 'aws configure set aws_secret_access_key $AWS_SECRET_KEY'
                    sh 'aws configure set region us-east-1' 
                    sh "aws s3 cp target/${JARNAME} s3://lambda-tstbucket/lambda-test/"


                    sh "aws lambda update-function-code --function-name testfunction  --zip-file fileb://target/${JARNAME}"

                }          
            }
        }

        stage('Release to Prod') {
            agent none
            steps {
                echo 'Release to Prod'
                script {
                    if (env.BRANCH_NAME == "master") {
                        timeout(time: 1, unit: 'HOURS') {
                            input('Proceed for Prod  ?')
                        }
                    }
                }

            }
        }

         stage('Deploy to Prod') {
            agent any
            steps {
                script {
                    if (env.BRANCH_NAME == "master") {
                        echo 'Deploy to Prod'
                        echo "ARTIFACTID: ${ARTIFACTID}"
                        echo "VERSION: ${VERSION}"
                        JARNAME = ARTIFACTID+'-'+VERSION+'.jar'

                        sh "aws s3 cp target/${JARNAME} s3://lambda-tstbucket/lambda-prod/"
                        //  sh './deploy-test.sh $AWS_ACCESS_KEY $AWS_SECRET_KEY'
                        // if (does_lambda_exist('prodfunction')) {
                            sh "aws lambda update-function-code --function-name prodfunction --s3-bucket lambda-tstbucket --s3-key lambda-prod/${JARNAME}"
                        //}  
                    }
                }
            }
        }

    }

    post {
      failure {
        echo 'failed'
             mail to: 'nirmala16.psg@gmail.com',
             subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
             body: "Something is wrong with ${env.BUILD_NUMBER}"
      }
      success {
        echo 'Success'
      }
      aborted {
        echo 'aborted'
      }
    }
}

def does_lambda_exist(String name) {	
  isexist=false
  echo $name
  try{
    sh  'aws lambda get-function --function-name test'
    isexist=true
  }
  catch(Exception e) {
    echo 'build Failed'
    isexist=true
  }
  return isexist
}
