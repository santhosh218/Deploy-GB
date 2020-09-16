pipeline {
	agent any
	stages {

		stage('Lint Blue server HTML') {
			steps {
				sh 'tidy -q -e ./Blue/*.html'
			}
		}

        	stage('Build Blue Docker Image') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh "docker build --tag=udacitybluecap ./Blue/"
				}
			}
		}

		stage('Push Blue Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'dockerhub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh 'docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD'
					sh 'docker tag udacitybluecap nakotisanthosh/udacitybluecap'
					sh 'docker push nakotisanthosh/udacitybluecap'
				}
			}
		}

        	stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:142977788479:cluster/CapstoneCluster
					'''
				}
			}
		}

        	stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl create -f ./Blue/blue-controller.yml
					'''
				}
			}
		}

        	stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl create -f ./Blue/blue-service.yml
					'''
				}
			}
		}

		stage('Wait user approve') {
			steps {
				input "Is that Blue server is working then approve for Green?"
            }
        }

		stage('Creating storage class') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl create -f ./Green/greensc.yml
					'''
				}
			}
		}


		stage('Creating persistent volume') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl create -f ./Green/greenpv.yml
					'''
				}
			}
		}

		stage('Creating persistent volume claim') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl create -f ./Green/greenpvc.yml
					'''
				}
			}
		}

		stage('Deploy Green container') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl create -f ./Green/green-controller.yml
					'''
				}
			}
		}

		stage('Deploy Green service') {
			steps {
				withAWS(region:'us-west-2', credentials:'aws-static') {
					sh '''
						kubectl create -f ./Green/green-service.yml
					'''
				}
			}
		}


    }
}
