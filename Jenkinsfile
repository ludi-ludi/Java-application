pipeline {
    agent any

    environment {
		DOCKERHUB_CREDENTIALS=credentials('docker-cred')
	}



    


    stages {
        stage('clean env') {
            steps {
                sh '''
            docker system prune -fa || true
                '''
            }
        }
        stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}        
        
        stage('compile') {
            steps {
               sh 'mvn compile'
            }
        }

        



        stage('Test') {
            steps {
                sh "mvn test  -DskipTests=true"
            }
        }


        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }

    //    stage('SonarQube analysis') {
    //        agent {
    //            docker {
    //              image 'sonarsource/sonar-scanner-cli:4.8.0'
    //            }
    //           }
    //           environment {
    //    CI = 'true'
    //    scannerHome='/opt/sonar-scanner'
    //    }
    //        steps{
    //            withSonarQubeEnv('sonar') {
    //                sh "${scannerHome}/bin/sonar-scanner"
    //            }
    //    }
    stage('Build') {
            steps {
               sh "mvn package -DskipTests=true "
            }
    
    }
    stage('Build-images') {
            steps {
                sh '''
              docker build -t  fridade/comm-card:jenkins-$BUILD_NUMBER .

                '''
            }

    }
    stage('Docker Image Scan') {
            steps {
                sh "trivy image --format table -o trivy-image-report.html fridade/comm-card:jenkins-$BUILD_NUMBER "
            }
        }

   

    stage('Push-ui') {
          
            steps {
               sh 'docker push fridade/comm-card:jenkins-$BUILD_NUMBER'
            }
    }
    stage('helm-charts') {


	      steps {
	        script {
	          withCredentials([
	            string(credentialsId: 'github-cred', variable: 'TOKEN')
	          ]) {

	            sh '''
                rm -rf commercial-card || true 
                git clone https://$TOKEN@github.com/fridade/commercial-card.git
                cd commercial-card
cat << EOF > values.yaml
       repository:
         tag:   jenkins-$BUILD_NUMBER
         assets:
          image: fridade/comm-card
       
         
EOF
git config --global user.name "fridade"
git config --global user.email "info@fridade.com"
   cat  values.yaml
   
   git add -A 
    git commit -m "Change from JENKINS" 
    git push  https://fridade:$TOKEN@github.com/fridade/commercial-card.git
	            '''
	          }

	        }

	      }

	    }


      




       
        





   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   
   }































}
