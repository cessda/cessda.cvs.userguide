/*
Copyright CESSDA ERIC 2017-2019

Licensed under the Apache License, Version 2.0 (the "License"); you may not
use this file except in compliance with the License.
You may obtain a copy of the License at
http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
pipeline {
	options {
		buildDiscarder logRotator(numToKeepStr: '20')
	}

	environment {
		productName = 'cvs'
		componentName = 'userguide'
		imageTag = "${docker_repo}/${productName}-${componentName}:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
	}

	agent any

	stages {
		// Compiles documentation
		stage('Build Documentation') {
			agent {
				dockerfile {
					filename 'jekyll.Dockerfile'
					reuseNode true
				}
			}
			stages {
				stage('Lint Documentation') {
					steps {
						sh "bundle exec rake lint"
					}
				}
				stage('Build Deployable Documentation') {
					steps {
						sh "jekyll build"
						// sh "bundle exec rake htmlproofer"
					}
				}
			}
		}
		stage('Build Nginx Container') {
			steps {
				sh "docker build -t ${imageTag} -f nginx.Dockerfile ."
			}
			when { branch 'main' }
		}
		stage('Push Docker Container') {
			steps {
				sh "gcloud auth configure-docker"
				sh "docker push ${imageTag}"
				sh "gcloud container images add-tag ${imageTag} ${docker_repo}/${productName}-${componentName}:${env.BRANCH_NAME}-latest"
			}
			when { branch 'main' }
		}
		stage('Deploy Nginx Container') {
			steps {
				build job: 'cessda.cvs.deploy/main', parameters: [string(name: 'userguide_image_tag', value: "${env.BRANCH_NAME}-${env.BUILD_NUMBER}")], wait: false
			}
			when { branch 'main' }
		}
	}
}
