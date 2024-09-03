// pipeline {
//     agent any

//     environment {
//         DOCKERHUB_CREDENTIALS = credentials('del-docker-hub-auth')
//     }

//     options {
//         buildDiscarder(logRotator(numToKeepStr: '5'))
//         disableConcurrentBuilds()
//         timeout(time: 60, unit: 'MINUTES')
//         timestamps()
//     }

//     stages {
//         stage('Clean env') {
//             steps {
//                sh '''
//             docker system prune -fa || true
//                   ''' 
//             }
//         }

//         stage('Checkout') {
//             steps {
//                 git branch: 'main', url: 'https://github.com/ludi-ludi/Java-application.git'
//             }
//         }

//         stage('Login to DockerHub') {
//             steps {
//                 sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
//             }
//         }

//         stage('Compile') {
//             agent {
//                 docker {
//                     image 'maven:3.8.5-openjdk-17'
//                     args '-v $HOME/.m2:/root/.m2' // Caches Maven dependencies between builds
//                 }
//             }
//             steps {
//                 sh 'mvn clean compile'
//             }
//         }

//         stage('Test') {
//             agent {
//                 docker {
//                     image 'maven:3.8.5-openjdk-17'
//                     args '-v $HOME/.m2:/root/.m2' // Caches Maven dependencies between builds
//                 }
//             }
//             steps {
//                 echo 'Running tests...'
//                 sh 'mvn test -DskipTests=true'  // Tests are skipped here
//             }
//         }

//         stage('SonarQube Analysis') {
//             agent {
//                 docker {
//                     image 'sonarsource/sonar-scanner-cli:5.0.1'
//                 }
//             }
//             environment {
//                 CI = 'true'
//                 scannerHome = '/opt/sonar-scanner'
//             }
//             steps {
//                 withSonarQubeEnv('Sonar') {
//                     sh "${scannerHome}/bin/sonar-scanner"
//                 }
//             }
//         }

//         stage('Package') {
//             agent {
//                 docker {
//                     image 'maven:3.8.5-openjdk-17'
//                 }
//             }
//             steps {
//                 sh 'mvn clean package -DskipTests=true'
//                 stash includes: 'target/*.jar', name: 'jar'
//             }
//         }

//         stage('Build and Push Docker Image') {
//             steps {
//                 unstash 'jar'
//                 script {
//                     def imageName = "devopseasylearning/s5ludivine:javaapp-$BUILD_NUMBER"
                    
//                     // Ensure the artifact exists
//                     sh 'ls -l target/*.jar'

//                     // Build Docker image
//                     sh """
//                     docker build -t $imageName .
//                     docker push $imageName
//                     """
//                 }
//             }
//         }

//         stage('deploy with docker') {
//             steps {
//                 sh 'docker run -itd -p 1973:8080 devopseasylearning/s5ludivine:javaapp-$BUILD_NUMBER'
//             }
//         }

//         stage('retrieve the public ip address') {
//             steps {
//                 sh 'curl ifconfig.io'
//             }
//         }

//     }
// }



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
        stage('Clean env') {
            steps {
               sh '''
                docker system prune -fa || true
               ''' 
            }
        }

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
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                    args '-v $HOME/.m2:/root/.m2' // Caches Maven dependencies between builds
                }
            }
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('Test') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                    args '-v $HOME/.m2:/root/.m2' // Caches Maven dependencies between builds
                }
            }
            steps {
                echo 'Running tests...'
                sh 'mvn test -DskipTests=true'  // Tests are skipped here
            }
        }

        stage('SonarQube Analysis') {
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

        stage('Package') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-17'
                }
            }
            steps {
                sh 'mvn clean package -DskipTests=true'
                stash includes: 'target/*.jar', name: 'jar'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                unstash 'jar'
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

        stage('Deploy with Docker') {
            steps {
                sh 'docker run -itd -p 1973:8080 devopseasylearning/s5ludivine:javaapp-$BUILD_NUMBER'
            }
        }

        stage('Retrieve the Public IP Address') {
            steps {
                sh 'curl ifconfig.io'
            }
        }

        // Helm Chart Management Stage
        stage('Helm Charts') {
            steps {
                script {
                    sh """
                    rm -rf java-application || true
                    git clone https://github.com/ludi-ludi/java-application.git
                    cd java-application

                    cat <<EOF > values.yaml
                    image:
                      repository: devopseasylearning/s5ludivine
                      tag: javaapp-$BUILD_NUMBER
                      pullPolicy: IfNotPresent
     
                    EOF

                    git config --global user.name "ludi-ludi"
                    git config --global user.email "ludi@git.com"
                    git add values.yaml
                    git commit -m "Update Helm values from Jenkins build"
                    git push https://github.com/ludi-ludi/java-application.git
                    """
                }
            }
        }

        // Helm Deployment Stage
        // stage('Deploy with Helm') {
        //     steps {
        //         sh """
        //         helm upgrade --install my-java-app ./java-application \
        //             --values ./java-application/dev-values.yaml
        //         """
        //     }
        // }

    }
}
