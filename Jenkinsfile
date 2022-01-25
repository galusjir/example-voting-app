pipeline {
    agent any

    stages {
/*
        Worker pipeline
*/
        stage('Worker-Build') {
            agent {
                docker {
                    image   'maven:3.6.1-jdk-8-alpine'
                    args    '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset '**/worker/*'
            }
            steps {
                echo 'Compiling worker app'
                dir('worker') {
                    sh 'mvn compile' 
                }
            }
        }
        stage('Worker-Test') {
            agent {
                docker {
                    image   'maven:3.6.1-jdk-8-alpine'
                    args    '-v $HOME/.m2:/root/.m2'
                }
            }
            when {
                changeset '**/worker/*'
            }
            steps {
                echo 'Testing the worker'
                dir('worker') {
                    sh 'mvn clean test' 
                }
            }
        }
        
        stage('Worker-Package') {
            agent {
                docker {
                    image   'maven:3.6.1-jdk-8-alpine'
                    args    '-v $HOME/.m2:/root/.m2'
                }
            }
            
            when {
                changeset '**/worker/*'
            }
            steps {
                echo 'Packaging the project'
                dir('worker') {
                    sh 'mvn package -DSkipTests' 
                }
            }
        }

        stage('Worker-BuildContainer') {
            agent any
            when {
              changeset '**/worker/*'
              branch 'master'
            }
            steps {
                script {
                    docker.withRegistry("https://registry.hub.docker.com/v1/","dockerhub_token") {
                        def image = docker.build("galusjir/worker:${env.BUILD_ID}", "worker")
                        image.push()
                    }
                }
                
            }
        }


/*
        Jenkins Result application
*/

         stage('Result-Build') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }
            when {
                changeset '**/result/*'
            }
            steps {
                echo 'Compiling voting app'
                dir('result') {
                    sh 'npm install' 
                }
            }
        }
        stage('Result-Test') {
            agent {
                docker {
                    image 'node:8.16.0-alpine'
                }
            }

            when {
                changeset '**/result/*.js'
            }
            
            steps {
                echo 'Testing voting app'
                dir('result') {
                    sh 'npm install'
                    sh 'npm test'
                }
            }
        }

         stage('Result-BuildContainer') {
            agent any
            when {
              changeset '**/Result/*'
              branch 'master'
            }
            steps {
                script {
                    docker.withRegistry("https://registry.hub.docker.com/v1/","dockerhub_token") {
                        def image = docker.build("galusjir/result:${env.BUILD_ID}", "result")
                        image.push()
                        image.push('latest')
                    }
                }
                
            }
        }

/*
       Vote application
*/
        stage('Vote-Build') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }
            when {
                changeset '**/vote/*'
            }
            steps {
                echo 'Compiling voting app'
                dir('vote') {
                    sh 'pip install -r requirements.txt' 
                }
            }
        }

        stage('Vote-Test') {
            agent {
                docker {
                    image 'python:2.7.16-slim'
                    args '--user root'
                }
            }            
            when {
                changeset '**/vote/*'
            }
            
            steps {
                echo 'Testing voting app'
                dir('vote') {
                    sh 'pip install -r requirements.txt'
                    sh 'nosetests -v'
                }
            }
        }

        stage('Vote-BuildContainer') {
            agent any

            when {
                changeset '**/vote/*'
//		        changeRequest()   // This would run on every change request
                branch 'master'
            }
            
            steps {
                echo 'Building Vote container'
                script {
                    docker.withRegistry("https://registry.hub.docker.com/v1/","dockerhub_token") {
                        def image = docker.build("galusjir/vote:${env.BUILD_ID}", "vote")
                        image.push()
                    }
                }
                
            }
        }

// And If all is good, then you can docker compose everything up
        stage("Deliver to dev") {
            agent any
            when{
                // When at least 1 container was created
            }
            script {
                sh 'docker-compose up -d'
            }
        }
        
    }
}
