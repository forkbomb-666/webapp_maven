pipeline {
	environment {
		registry = "arig23498/webapp"
		registryCredential = 'dockerhubid'
		dockerImage = ''
	}
	agent any
	stages {
		stage('SCM Checkout') {
			steps{
				git 'https://github.com/forkbomb-666/webapp_maven'
			}
		}
		stage('Compile-Package') {
			steps {
				script{
					def mvn_home = tool name: 'maven-3.6.0', type: 'maven'
					sh "${mvn_home}/bin/mvn package"
				}
			}
		}
		stage('Building Image') {
			steps {
				script {
					dockerImage = docker.build registry + ":$BUILD_NUMBER"
				}
			}
		}
		stage('Deploy Image') {
			steps {
				script {
					docker.withRegistry('', registryCredential) {
						dockerImage.push()
					}
				}
			}
		}
		stage('Remove Unused docker image') {
			steps {
				script {
					sh "docker rmi $registry:$BUILD_NUMBER"
				}
			}
		}
		stage('Deploy Application') {
			steps {
				script {
					sh '''if kubectl get deployments.apps | grep -q webapp-v;
then
	kubectl delete deployment webapp-v`expr $BUILD_NUMBER - 1`;
fi'''
					sh "kubectl run --image=arig23498/webapp webapp-v$BUILD_NUMBER --port=9090"
				}
			}
		}
		stage('Expose Application') {
			steps {
				script {
					sh '''if kubectl get services | grep -q webapp-v;
then
	kubectl delete service webapp-v`expr $BUILD_NUMBER - 1`;
fi'''
					sh "kubectl expose deployment webapp-v$BUILD_NUMBER --type=LoadBalancer --port=9090 --target-port=9090 --external-ip=104.211.230.185"
				}
			}
		}
	}
}
