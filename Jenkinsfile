pipeline {
    agent any

    environment {
        DOCKER_ID = 'peraldi'
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Build Images (Docker Compose)') {
            steps {
                sh 'docker-compose -f docker-compose.yml build'
            }
        }

        stage('Run Stack (Tests)') {
            steps {
                sh '''
                  docker-compose -f docker-compose.yml up -d
                  docker-compose ps
                '''
            }
        }

        stage('Docker Push') {
            environment {
                DOCKER_PASS = credentials('DOCKER_HUB_PASS')
            }
            steps {
                sh '''
                  docker login -u $DOCKER_ID -p $DOCKER_PASS

                  docker tag exam_jenkins_cast_service $DOCKER_ID/cast-service:$DOCKER_TAG
                  docker tag exam_jenkins_movie_service $DOCKER_ID/movie-service:$DOCKER_TAG

                  docker push $DOCKER_ID/cast-service:$DOCKER_TAG
                  docker push $DOCKER_ID/movie-service:$DOCKER_TAG
                '''
            }
        }

        stage('Cleanup Docker Compose') {
            steps {
                sh 'docker-compose down'
            }
        }

        stage('Deploiement en Dev') {
            environment {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                sh '''
                  rm -Rf .kube
                  mkdir .kube
                  ls
                  cat $KUBECONFIG > .kube/config
                  cp charts/values.yaml values.yml
                  cat values.yml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                  helm upgrade --install app-dev charts \
                  --namespace dev \
                  --create-namespace
                '''
            }
        }

        stage('Deploiement en QA') {
            environment {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                sh '''
                  rm -Rf .kube
                  mkdir .kube
                  ls
                  cat $KUBECONFIG > .kube/config
                  cp charts/values.yaml values.yml
                  cat values.yml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                  helm upgrade --install app-qa charts \
                  --namespace qa \
                  --create-namespace
                '''
            }
        }

        stage('Deploiement en Staging') {
            environment {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                sh '''
                  rm -Rf .kube
                  mkdir .kube
                  ls
                  cat $KUBECONFIG > .kube/config
                  cp charts/values.yaml values.yml
                  cat values.yml
                  sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                  helm upgrade --install app-staging charts \
                  --namespace staging \
                  --create-namespace
                '''
            }
        }

        stage('Deploiement en Production') {
            environment {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            when {
                expression { env.BRANCH_NAME == 'master' }
            }
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    sh '''
                      rm -Rf .kube
                      mkdir .kube
                      ls
                      cat $KUBECONFIG > .kube/config
                      cp charts/values.yaml values.yml
                      cat values.yml
                      sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml
                      helm upgrade --install app-prod charts \
                      --namespace prod \
                      --create-namespace
                    '''
                }
            }
        }
    }
}
