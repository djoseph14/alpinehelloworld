pipeline {
    
    environment { 
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "ajc-2.1"
        STAGING = "daniel-ajc-staging-env"
        PRODUCTION = "daniel-ajc-prod-env"
        USERNAME = "lianhuahayu"
        CONTAINER_NAME = IMAGE_NAME
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

        
    }

}