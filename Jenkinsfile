pipeline {
	agent any
	triggers {
    		pollSCM 'H/12 * * * *'
    	}
		stages 
		{
			stage("Deliver to Docker Hub")
			{
				steps
				{
					sh "docker build . -t gruppe1devops/frontend-calc"
					withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'DockerID', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']])
					{
						sh 'docker login -u ${USERNAME} -p ${PASSWORD}'
					}
					sh "docker push gruppe1devops/frontend-calc"
				}
			}
			
			stage("Selenium grid setup")
			{
				steps
				{
					sh "docker network create Group1"
					sh "docker run -d --rm -p 11111:4444 --net=Group1 --name selenium-hub-Group1 selenium/hub"
					sh "docker run -d --rm --net=Group1 -e HUB_HOST=selenium-hub-Group1 --name selenium-node-firefox-Group1 selenium/node-firefox" 
					sh "docker run -d --rm --net=Group1 -e HUB_HOST=selenium-hub-Group1 --name selenium-node-chrome-Group1 selenium/node-chrome"
					sh "docker run -d --rm --net=Group1 --name app-test-container-Group1 math231k/frontend-calc"
				}
			}
			
			stage("Execute system tests") 
			{
				steps 
				{
					sh "selenium-side-runner --server http://localhost:11111/wd/hub -c 'browserName=firefox' --base-url http://app-test-container-Group1 test/system/FrontendCalculatortest.side"
					sh "selenium-side-runner --server http://localhost:11111/wd/hub -c 'browserName=chrome' --base-url http://app-test-container-Group1 test/system/FrontendCalculatortest.side"
				}				
			}
		}
		post 
		{
			cleanup 
			{
				echo "Cleaning the Docker environment"
				sh script:"docker stop selenium-hub-Group1", returnStatus:true
				sh script:"docker stop selenium-node-firefox-Group1", returnStatus:true
				sh script:"docker stop selenium-node-chrome-Group1", returnStatus:true
				sh script:"docker stop app-test-container-Group1", returnStatus:true
				sh script:"docker network remove Group1", returnStatus:true
			}
		}
	}
