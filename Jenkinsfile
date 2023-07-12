pipeline {
environment { // Declaration of environment variables
DOCKER_ID = "elfiadylclc" // replace this with your docker-id
DOCKER_IMAGE_MOVIE = "datascientestmovie"
DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build
}
agent any // Jenkins will be able to select all available agents
stages {

        stage(' Docker run movie db '){   
             steps {
                script {
                sh '''
                docker rm -f movie_db
                docker run -d --volume postgres_data_movie:/var/lib/postgresql/data/ -e POSTGRES_USER='movie_db_username' -e POSTGRES_PASSWORD='movie_db_password' -e POSTGRES_DB='movie_db_dev' --name movie_db postgres:12.1-alpine
                sleep 10
                '''
                }
              }
        }    

        stage(' Docker Build movie'){

                    steps {
                        script {
                        sh '''
                        docker rm -f movie_service
                        docker build -t $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG ./movie-service/
                        sleep 6
                        '''
                        }
                    }
        }

         stage(' Docker run movie'){
           
                        steps {
                            script {
                            sh '''
                            docker run -d -p 8001:8000 --volume ./movie-service/:/app/ -e DATABASE_URI='postgresql://movie_db_username:movie_db_password@movie_db/movie_db_dev' --name movie_service $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG 
                            sleep 10
                            '''
                            }
                        }
        }    

        stage('Test Acceptance'){ // we launch the curl command to validate that the container responds to the request
            steps {
                    script {
                    sh '''
                    curl localhost:8001
                    '''
                    }
            }

        }
        stage('Docker Push'){ //we pass the built image to our docker hub account
            environment
            {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve  docker password from secret text called docker_hub_pass saved on jenkins
            }

            steps {

                script {
                sh '''
                docker login -u $DOCKER_ID -p $DOCKER_PASS
                docker push $DOCKER_ID/$DOCKER_IMAGE_MOVIE:$DOCKER_TAG
                '''
                }
            }

        }
        
        stage('Deploiement db en dev'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp moviedb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install moviedb moviedb --values=values.yml --namespace dev
                                '''
                                }
                            }

         }
      

        stage(' Deploiement app en dev'){

                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp movie/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install movie movie --values=values.yml --namespace dev
                                '''
                                }
                            }

        }                 

        stage(' Deploiement db en qa'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp moviedb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install moviedb moviedb --values=values.yml --namespace qa
                                '''
                                }
                            }

        }
		

        stage(' Deploiement app en qa'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp movie/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install movie movie --values=values.yml --namespace qa
                                '''
                                }
                            }

        }

        stage(' Deploiement movie db en staging'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp moviedb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install moviedb moviedb --values=values.yml --namespace staging
                                '''
                                }
                            }
        }        

        stage(' Deploiement en staging'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp movie/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install movie movie --values=values.yml --namespace staging
                                '''
                                }
                            }

                
        }

        stage(' Deploiement movie db en prod'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                timeout(time: 15, unit: "MINUTES") {
                                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                                }
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp moviedb/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install moviedb moviedb --values=values.yml --namespace prod
                                '''
                                }
                            }

        }

        stage(' Deploiement movie en prod'){
                        environment
                        {
                        KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
                        }
                            steps {
                                timeout(time: 15, unit: "MINUTES") {
                                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                                }
                                script {
                                sh '''
                                rm -Rf .kube
                                mkdir .kube
                                ls
                                cat $KUBECONFIG > .kube/config
                                cp movie/values.yaml values.yml
                                cat values.yml
                                sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                                helm upgrade --install movie movie --values=values.yml --namespace prod
                                '''
                                }
                            }

        }
}
}
