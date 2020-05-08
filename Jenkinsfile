pipeline {

    agent none

    environment {

        NODE_ENV="homolog"
        AWS_ACCESS_KEY=""
        AWS_SECRET_ACCESS_KEY=""
        AWS_SDK_LOAD_CONFIG="0"
        BUCKET_NAME="dh-pi-devops-dev-karen"
        REGION="us-east-1" 
        PERMISSION=""
        ACCEPTED_FILE_FORMATS_ARRAY=""
        VERSION="1.0.0"
    }


    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
    }
    triggers {
        cron('@daily')
    }

    stages{
        stage("Build, Test and Push Docker Image") {
            agent {  
                node {
                    label 'master'
                }
            }
            stages {

                stage('Clone repository') {
                    steps {
                        script {
                            if(env.GIT_BRANCH=='origin/master'){
                                checkout scm
                            }
                            sh('printenv | sort')
                            echo "My branch is: ${env.GIT_BRANCH}"
                        }
                    }
                }
                stage('Build image'){       
                    steps {
                        script {
                            print "Environment will be : ${env.NODE_ENV}"
                            docker.build("digitalhouse-devops-app:latest")
                        }
                    }
                }

                stage('Test image') {
                    steps {
                        script {

                            docker.image("digitalhouse-devops-app:latest").withRun('-p 3000:3000') { c ->
                                sh 'docker ps'
                                sh 'sleep 10'
                                sh 'curl http://127.0.0.1:3000/api/v1/healthcheck'
                                
                            }
                    
                        }
                    }
                }

                stage('Docker push') {
                    steps {
                        echo 'Push latest para AWS ECR'
                        script {
                            docker.withRegistry('https://893817927886.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ecraws') {
                                docker.image('digitalhouse-devops-app').push()
                            }
                        }
                    }
                }
            }
        }

        stage('Deploy to Homolog') {
            agent {  
                node {
                    label 'homol'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/master'){
 
                        docker.withRegistry('https://893817927886.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ecraws') {
                            docker.image('digitalhouse-devops-app').pull()
                        }

                        echo 'Deploy para Homologação'
                        sh "hostname"
                        catchError {
                            sh "docker stop app_homolog"
                            sh "docker rm app_homolog"
                        }
                        sh "docker run -d --name app_homolog -p 3000:3000 893817927886.dkr.ecr.us-east-1.amazonaws.com/digitalhouse-devops-app:latest"
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:3000/api/v1/healthcheck'

                    }
                }
            }

        }

        stage('Deploy to Producao') {
            agent {  
                node {
                    label 'prod'
                }
            }

            steps { 
                script {
                    if(env.GIT_BRANCH=='origin/master'){

                        docker.withRegistry('https://893817927886.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ecraws') {
                            docker.image('digitalhouse-devops-app').pull()
                        }

                        echo 'Deploy para Produção'
                        sh "hostname"
                        catchError {
                            sh "docker stop app_prod"
                            sh "docker rm app_prod"
                        }
                        sh "docker run -d --env NODE_ENV=producao --env BUCKET_NAME=dh-pi-devops-prod-karen --name app_prod -p 80:3000 893817927886.dkr.ecr.us-east-1.amazonaws.com/digitalhouse-devops-app:latest"
                        sh "docker ps"
                        sh 'sleep 10'
                        sh 'curl http://127.0.0.1:80/api/v1/healthcheck'

                    }
                }
            }

        }

    }
}
