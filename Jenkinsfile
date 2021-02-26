pipeline {
	agent any
		stages 
		{
			stage("Deliver to Docker Hub")
			{
				steps
				{
					sh "docker build . -t math231k/frontend-calc"
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerID', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
					{
						sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
					}
					sh "docker push math231k/frontend-calc"
				}
			}
			
			stage("Selenium grid setup")
			{
				steps
				{
					sh "docker network create SE"
					sh "docker run -d --rm -p 4444:11111 --net=SE --name selenium-hub selenium/hub"
					sh "docker run -d --rm --net=SE -e HUB_HOST=selenium-hub --name selenium-node-firefox selenium/node-firefox" 
					sh "docker run -d --rm --net=SE -e HUB_HOST=selenium-hub --name selenium-node-chrome selenium/node-chrome"
					sh "docker run -d --rm --net=SE --name app-test-container math231k/frontend-calc"
				}
			}
			
			stage("Execute system tests") 
			{
				steps 
				{
					sh "selenium-side-runner --server http:localhost:11111/wd/hub -c 'browserName=firefox' --base-url http://app-test-container test/system/FrontendCalculatortest.side"
					sh "selenium-side-runner --server http:localhost:11111/wd/hub -c 'browserName=chrome' --base-url http://app-test-container test/system/FrontendCalculatortest.side"
				}				
			}
		}
		post 
		{
			cleanup 
			{
				echo "Cleaning the Docker environment"
				sh script:"docker stop selenium-hub", returnStatus:true
				sh script:"docker stop selenium-node-firefox", returnStatus:true
				sh script:"docker stop selenium-node-chrome", returnStatus:true
				sh script:"docker stop selenium-app-test-container", returnStatus:true
				sh script:"docker network remove SE", returnStatus:true
			}
		}
	}
