pipeline {
    agent any

    environment {
        DOCKER_ID = 'peraldi'
        DOCKER_IMAGE_TAG = 'latest'
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
        HELM_CHART_PATH = 'charts'
    }

    stages {

        stage('Build Images (Docker Compose)') {
            steps {
                sh 'docker-compose -f $DOCKER_COMPOSE_FILE build'
            }
        }

        stage('Run Stack (Tests)') {
            steps {
                sh '''
                  docker-compose -f $DOCKER_COMPOSE_FILE up -d
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

                  docker tag exam_jenkins_cast_service $DOCKER_ID/cast-service:$DOCKER_IMAGE_TAG
                  docker tag exam_jenkins_movie_service $DOCKER_ID/movie-service:$DOCKER_IMAGE_TAG

                  docker push $DOCKER_ID/cast-service:$DOCKER_IMAGE_TAG
                  docker push $DOCKER_ID/movie-service:$DOCKER_IMAGE_TAG
                '''
            }
        }

        stage('Cleanup Docker Compose') {
            steps {
                sh 'docker-compose down'
            }
        }

        stage('Deploy to Dev (Auto)') {
            environment {
            KUBECONFIG = credentials("config") // we retrieve  kubeconfig from secret file called config saved on jenkins
            }
            steps {
                sh '''
                  rm -Rf .kube
                  mkdir .kube
                  ls
                  cat $KUBECONFIG > .kube/config
                  helm upgrade --install app-dev $HELM_CHART_PATH \
                  --namespace dev \
                  --create-namespace
                '''
            }
        }

        stage('Deploy to QA (Auto)') {
            steps {
                sh '''
                  helm upgrade --install app-qa $HELM_CHART_PATH \
                  --namespace qa \
                  --create-namespace
                '''
            }
        }

        stage('Deploy to Staging (Auto)') {
            steps {
                sh '''
                  helm upgrade --install app-staging $HELM_CHART_PATH \
                  --namespace staging \
                  --create-namespace
                '''
            }
        }

        stage('Deploy to Production (Manual)') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
                    input message: 'Do you want to deploy in production ?', ok: 'Yes'
                    sh '''
                      helm upgrade --install app-prod $HELM_CHART_PATH \
                      --namespace prod \
                      --create-namespace
                    '''
                }
            }
        }
    }
}
