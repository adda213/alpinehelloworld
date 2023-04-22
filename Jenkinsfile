@Library(/adda213-share-library')_
pipeline {
     environment {
       IMAGE_NAME = "alpinehelloworld" 
       IMAGE_TAG = "latest"
       STAGING = "adda213-staging"
       PRODUCTION = "adda213-production"
     }
     agent none
     stages {
         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t adda213/$IMAGE_TAG .'  
                }
             }
         }
         stage('run container based on build image') {
             agent any
             steps {
                script {
                  sh '''
                     docker run --name $IMAGE_NAME -d -p 80:5000 -e PORT=5000 adda213/$IMAGE_NAME:$IMAGE_TAG
                     sleep 5
                  '''  
                }
             }
         }
         stage('Test image') {
             agent any
             steps {
                script {
                  sh '''
                     curl http://192.168.56.16 | grep -q "Hello world!"
                  '''  
                }
             }
         }
         stage('Clean Container') {
             agent any
             steps {
                script {
                  sh '''
                     docker stop $IMAGE_NAME
                     docker rm -f $IMAGE_NAME
                  '''  
                }
             }
         }
         stage('push image in staging and deploy it') {
             when {
                         expression { GIT_BRANCH == 'origin/master' }
             }
             agent any
             environment {
                 HEROKU_API_KEY = credentials('heroku_api_key')
             }
             steps {   script {
                  sh '''
                     docker login --username=king.of.net.adda@gmail.com --password=${HEROKU_API_KEY} registry.heroku.com
                     heroku create $STAGING || echo "project already exist"
                     heroku container:push -a $STAGING web
                     heroku container:release -a $STAGING web
                  '''  
                }
             }

         }
         stage('push image in production and deploy it') {
             when {
                         expression { GIT_BRANCH == 'origin/master' }
             }
             agent any
             environment {
                 HEROKU_API_KEY = credentials('heroku_api_key')
             }
             steps {   script {
                  sh '''
                     heroku container: login
                     heroku create $PRODUCTION || echo "project already exist"
                     heroku container:push -a $PRODUCTION web
                     heroku container:release -a $PRODUCTION web
                  '''  
                }
             }

         }

     }
  post {
       always {
           script { 
                slackNotifier currentBuild.result
            }
       }
  }
}

       
