pipeline {
	
	agent any

	environment {
        APP_NAME= "c-api"
	IMAGE_NAME= 'ohadd306/c-api'
	IMAGE_TAG="${BUILD_NUMBER}"

	}

	stages {
	
	    stage('checkout') {
		steps {
		   checkout scm

                }

		}
	    stage('tests') {
		steps {
		   echo "running python tests"
                   sh '''
                   python3 -m pytest -v
                ''' 
		}
	               }
            stage('build docker iamge') {
		steps {
		   echo "build docker image"
		   sh '''
			docker build -t ${IMAGE_NAME}:${IMAGE_TAG}  .
		'''
		}
		}

	    stage('test docker image')  {
	       steps { 
		  echo "test docker image"
		  sh '''
			docker run -d --name ${APP_NAME}-test -p 8002:8000 ${IMAGE_NAME}:${IMAGE_TAG}
		'''

              echo 

"wait for app "
	      sleep 5
              echo "test application health"
              
                 echo "test docker image"
                 sh '''
			curl -f http://localhost:8001/health
                        curl -f http://localhost:8001/products/1
                        curl -f  -X POST  -H "Content-Type: application/json" -d '{"product_id":1,"quantity":3}' http://localhost:8001/orders/
		       
		 '''
         } 

		

	    post {
		always {
                   echo "remove docker container test"
                   sh ''' 
                       docker rm -f ${APP_NAME}-test  || true
                   '''  
		   }
		}
             
}


	stage ('security scan') {
		steps {
                    sh ''' 
			trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 ${IMAGE_NAME}:${IMAGE_TAG}
		'''
		}

	}

// push docker image

        stage('Push to Docker Hub') {

            steps {

                echo "Logging into Docker Hub..."

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "$DOCKER_PASSWORD" | \
                        docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        docker push \
                            ${IMAGE_NAME}:${IMAGE_TAG}

                        docker logout
                    '''
                }
            }
        }




	stage ('deploy on stg') {
	   steps {
	      echo "deploy on stg"
	      sh '''
	        docker stop ${APP_NAME}-stg || true
		docker rm  ${APP_NAME}-stg || true
		docker pull ${IMAGE_NAME}:${IMAGE_TAG}
		docker run -d  --name ${APP_NAME}-stg -p 8001:8000  ${IMAGE_NAME}:${IMAGE_TAG}
	''' 	
	}



	}

          echo "wait for app to start"
          sleep 5 
          echo "test stg application"
          
	  sh '''
             curl -f http://localhost:8001/health
             curl -f http://localhost:8001/products/1
             curl - f -X post -H "Content-Type: application/json" -d '{"product_id":1,"quantity":3}' http://localhost:8001/orders

         


          '''


	}
}

