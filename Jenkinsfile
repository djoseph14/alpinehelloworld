pipeline {
    
    environment { 
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "ajc-2.1"
        STAGING = "daniel-ajc-staging-env"
        PRODUCTION = "daniel-ajc-prod-env"
        USERNAME = "lianhuahayu"
        CONTAINER_NAME = "alpinehelloworld"
        EC2_PRODUCTION_HOST = "54.88.166.62"
    }

    agent none 

    stages{
        stage('Build Image') {
            agent any 
            steps { 
                script{
                    sh 'docker build -t $USERNAME/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Lancement du container') {
            agent any
            steps {
                script{
                    sh '''
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                        docker run --name $CONTAINER_NAME -d -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        sleep 15
                    '''

                }
            }
        }

        stage('Test du container') {
            agent any
            steps {
                script{
                    sh '''
                        curl http://localhost:5000 | grep -iq "Hello world!"
                    '''

                    
                }
            }
        }

        stage('Clean env and save artifact') {
            agent any
            environment{
                PASSWORD = credentials('dockerhub_password')
            }
            steps {
                script{
                    sh '''
                        docker login -u $USERNAME -p $PASSWORD
                        docker push $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                        docker rmi $USERNAME/$IMAGE_NAME:$IMAGE_TAG || true

                    '''
                }
            }
        }

        stage('Push image in staging and deploy it') {
            when{
                expression { GIT_BRANCH == 'origin/master' } 
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('tokenHeroku')
            }
            steps {
                script{
                    sh '''
                        heroku container:login
                        heroku create $STAGING || echo "project already exist"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                    '''
                }
            }
        }

        stage('Push image in production and deploy it') {
            when{
                expression { GIT_BRANCH == 'origin/master' } 
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('tokenHeroku')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $PRODUCTION || echo "project already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }

        stage('Deploy app on EC2-cloud Production') {
        when{
            expression { GIT_BRANCH == 'origin/master' }
        }
        agent any
        steps{
            withCredentials([sshUserPrivateKey(credentialsId: "ec2_prod_private_key", keyFileVariable: 'keyfile', usernameVariable: 'ubuntu')]) {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    script { 
                        timeout(time: 15, unit: "MINUTES") {
                            input message: 'Do you want to approve the deploy in production?', ok: 'Yes'
                            }
                        sh '''
                        ssh -o StrictHostKeyChecking=no -i ${keyfile} ${ubuntu}@${EC2_PRODUCTION_HOST} docker run --name $CONTAINER_NAME -d -e PORT=5000 -p 5000:5000 $USERNAME/$IMAGE_NAME:$IMAGE_TAG
                        '''
                        }
                    }
                }
            }
        }
        
    }

    post{
        success{
            slackSend (color: '#00FF00', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#FF0000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }

}