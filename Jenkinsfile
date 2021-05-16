pipeline {
	agent any
  environment {
        VERSION = 'latest'
        PROJECT = 'capstone-sample-app'
				IMAGE = "$PROJECT"
				ECRURL = "https://356782802290.dkr.ecr.us-west-2.amazonaws.com/$PROJECT"
				ECRURI = "356782802290.dkr.ecr.us-west-2.amazonaws.com/$PROJECT"
				ECRCRED = 'ecr:us-west-2:Temp_User_For_Learning'
  }
	stages {
		stage("Lint Dockerfile") {
			steps {
				sh "docker run --rm -i hadolint/hadolint:v1.17.5 < Dockerfile"
			}
		}
		stage('Build preparations') {
      steps {
        script {
            // calculate GIT lastest commit short-hash
            gitCommitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            shortCommitHash = gitCommitHash.take(7)
            // calculate a sample version tag
            VERSION = shortCommitHash
            // set the build display name
            currentBuild.displayName = "#${BUILD_ID}-${VERSION}"
        }
      }
		}
		stage('Docker build') {
		  steps {
			  script {
			    // Build the docker image using a Dockerfile
			    docker.build("$IMAGE")
			  }
      }
		}
		// Uploading Docker images into AWS ECR
		stage('Pushing to ECR') {
		  steps{  
			  script {
			    sh 'aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 356782802290.dkr.ecr.us-west-2.amazonaws.com'
			    sh 'docker push 356782802290.dkr.ecr.us-west-2.amazonaws.com/capstone-sample-app:latest'
				// sh("eval \$(aws ecr get-login --no-include-email | sed 's|https://||')")
                // docker.withRegistry(ECRURL, ECRCRED)
				// docker.withRegistry('https://356782802290.dkr.ecr.us-west-2.amazonaws.com','ecr:us-west-2:aws-instance-role') {docker.image(IMAGE).push("latest")}
         	  }
      }
		}    
		stage('K8S Deploy') {
		  steps {
				withAWS(credentials: 'Temp_User_For_Learning', region: 'us-west-2') {
					sh "aws eks --region us-west-2 update-kubeconfig --name UdacityCapStone-Cluster"
					// Configure deployment
					sh "kubectl apply -f k8s/deployment.yml"
					// Configure service for loadbalancing
					sh "kubectl apply -f k8s/service.yml"
					// Set created image to do a rolling update
					sh "kubectl set image deployments/$PROJECT $PROJECT=$ECRURI:$VERSION"
				}
      }
    }
  }
	post {
		always {
		    // make sure that the Docker image is removed
		    sh "docker rmi $IMAGE | true"
		}
	}
}
