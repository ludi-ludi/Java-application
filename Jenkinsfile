pipeline {
    agent any

   
    environment {
        DOCKERHUB_CREDENTIALS = credentials('del-docker-hub-auth')
    }

     options {
        buildDiscarder(logRotator(numToKeepStr: '5'))
        disableConcurrentBuilds()
        timeout(time: 60, unit: 'MINUTES')
        timestamps()
    }

    stages {
        
        stage('Checkout') {
        
            steps {
              git branch: 'main', url: 'https://github.com/ludi-ludi/Java-application.git' 
            }   
        }
        
        // stage('clean env') {
        //     steps {
        //         sh '''
        //     docker system prune -fa || true
        //         '''
        //     }
        // }

        stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}        
        
        stage('compile') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                }
            }
            steps {
               sh 'mvn compile'
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                }
            }
            steps {
                echo 'Running tests...'
                sh 'mvn clean'
                sh 'mvn test -DskipTests=true' 
                 
            }
        }
        stage('SonarQube analysis') {
            agent {
                docker {
                    image 'sonarsource/sonar-scanner-cli:5.0.1'
                }
            }
            environment {
                CI = 'true'
                scannerHome = '/opt/sonar-scanner'
            }
            steps {
                withSonarQubeEnv('Sonar') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
  
        stage('Build') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                }
            }
            steps {
               sh "mvn package -DskipTests=true "
            }
    
        }

        stage('Build and Push Docker Image') {
           steps {
                script {
                    def imageName = 'devopseasylearning/s5ludivine:javaapp-$BUILD_NUMBER'
                    sh "docker build -t $imageName ."
                    sh "docker push $imageName"
                }
            }
        }

   }
}

