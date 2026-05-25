@Library("Shared") _

pipeline {

    agent { label "ahad" }

    environment {
        DOCKER_USER = "ahadkhan021"
        DJANGO_IMAGE = "ahadkhan021/django-notes-app"
        NGINX_IMAGE = "ahadkhan021/django-notes-nginx"
    }

    stages {
        stage("HELLO"){
            steps{
                script{
                    hello()
                }
            }
        }

        stage("Clone Code") {
            steps {
                echo "Cloning Repository..."

                git branch: "main",
                    url: "https://github.com/Ahad0p/django-notes-app.git"
            }
        }

        stage("Clean Previous Deployment") {
            steps {
                sh '''
                    docker compose down || true

                    docker rm -f django_cont || true
                    docker rm -f nginx_cont || true
                    docker rm -f db_cont || true
                '''
            }
        }

        stage("Build Images") {
            steps {
                sh '''
                    docker compose build
                '''
            }
        }

        stage("Docker Login & Push") {

            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerHubCred',
                        usernameVariable: 'USER',
                        passwordVariable: 'PASS'
                    )
                ]) {

                    sh '''
                        echo $PASS | docker login \
                        -u $USER \
                        --password-stdin

                        docker tag django_app:latest ${DJANGO_IMAGE}:latest
                        docker push ${DJANGO_IMAGE}:latest

                        docker tag nginx:latest ${NGINX_IMAGE}:latest
                        docker push ${NGINX_IMAGE}:latest

                        docker logout
                    '''
                }
            }
        }

        stage("Deploy") {
            steps {

                sh '''
                    docker compose up -d
                '''
            }
        }

        stage("Verify Deployment") {
            steps {

                sh '''
                    sleep 20

                    docker ps

                    docker compose logs --tail=50
                '''
            }
        }
    }

    post {

        success {

            echo "Deployment Completed Successfully"
        }

        failure {

            sh '''
                docker compose logs || true
            '''

            echo "Deployment Failed"
        }

        always {

            echo "Pipeline Finished"
        }
    }
}
