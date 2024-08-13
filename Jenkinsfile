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

        stage('Login to DockerHub') {
            steps {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test -DskipTests=true'
            }
        }

        stage('Package') {
            steps {
                sh 'mvn package -DskipTests=true'
                // Archive the JAR file as a build artifact
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    def imageName = "devopseasylearning/s5ludivine:javaapp-$BUILD_NUMBER"
                    
                    // Ensure the artifact exists
                    sh 'ls -l target/*.jar'

                    // Build Docker image
                    sh """
                    docker build -t $imageName .
                    docker push $imageName
                    """
                }
            }
        }
    }
}
