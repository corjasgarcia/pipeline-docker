
pipeline {
  agent {
	  label 'docker'}
	  
  
  
	tools { 
		maven 'apache-maven-3.5.0' 
		}
	/*
	parameters{
		choice(choices: ['DEV', 'PROD'], description: '', name: 'ENVIRONMENT')
	}
	*/
	environment {
				
				PROJECT_URL="https://github.com/corjasgarcia/spring-petclinic.git"
				TEST_URL= "https://github.com/corjasgarcia/adop-cartridge-java-regression-tests.git"
				ENVIRONMENT_URL="https://github.com/corjasgarcia/nginx-configuration.git"
				SERVICE_NAME = "tomcat"
				APP_URL = "http://52.16.226.150:8888/petclinic"
				JMETER_TESTDIR = "jmeter_dir"
				IP = "52.16.226.150"
			}
		
    stages {
        stage('Clone') {
			
            steps {
                echo 'Cloning repository'
				
				dir('RepoOne') {
					git url: "${PROJECT_URL}"
					}
				dir('RepoTwo') {
					git url:  "${TEST_URL}"
					}
					/*
				dir('RepoThree') {
					git url:  "${ENVIRONMENT_URL}"
					}
					*/
			
            }
        }
		 stage('Build') {
			
            steps {
				dir('RepoOne'){
				
                echo 'Building with Maven'
			
				sh "./mvnw clean install -DskipTests"
				
				/*
				
				
				Tenemos el war en RepoOne/target..
				Vamos a crear una imagen de tomcat
				Siempre sera la misma, luego el contenedor será distinto
				*/
				sh "docker build -t mytomcat:last -f Dockerfile ."
				
				}
            }
        }
		stage ('Environment'){
			
		steps{
		
		dir(RepoThree){
		sh '''
			
				function createDockerContainer() {
							echo $1, $2
							#export ENVIRONMENT_NAME=$1
							#export SERVICE_NAME="$(echo ${PROJECT_NAME} | tr '/' '_')_${ENVIRONMENT_NAME}"
							#Creamos el nginx
							#No se levanta por algo de la carpeta que esta creando
							docker run --name mynginx -d -p 80:80 -v nginx_conf:/etc/nginx/conf.d nginx
							#levantariamos el contenedor tomcat
							## Add nginx configuration
							
							sed -i "s/###TOMCAT_SERVICE_NAME###/${SERVICE_NAME}/" $2
							docker cp $2 proxy:/etc/nginx/conf.d/${SERVICE_NAME}.conf
						}
							createDockerContainer "CI" tomcat.conf
			'''
		}
		}
		}

		
 
		/*
		stage('UnitTestJob'){
			
			steps{	
				sh "./mvnw clean test"
		}
		}
		stage('SonarQube analysis 1'){
	
		//Get properties from maven in the pom	
			steps{
				withSonarQubeEnv('sonarQube5.3') {
				sh './mvnw sonar:sonar'
		}
		}
	}
		stage('SonarQube analysis 2') {
			steps{
				script{
					scannerHome = tool 'SonarQube Scanner 2.8';
					}
				// requires SonarQube Scanner 2.8+
				withSonarQubeEnv('sonarQube5.3') {
				sh "${scannerHome}/bin/sonar-scanner"
		}
    }
  }
  */

		stage('agent Docker: Deployment'){
			
			
			steps{		
				dir("RepoOne"){
				sh '''
						#Tenemos la aplicacion ya en la imagen del contenedor
						#Debemos levantar el contenedor
					  docker images
					  #CONTAINER_ID = $(docker run -d --name ${SERVICE_NAME} -p 8888:8080 mytomcat:last)
					  COUNT=1
				      while ! curl -q http://52.16.226.150:8888/petclinic -o /dev/null
                      do
					  if [ ${COUNT} -gt 10 ]; then
                      echo "Docker build failed even after ${COUNT}. Please investigate."
                      exit 1
                      fi
                      echo "Application is not up yet. Retrying ..Attempt (${COUNT})"
                      sleep 5
                      COUNT=$((COUNT+1))
                      done'''
						
					
				}
				
			}
			
		}
		
	
		/*
		stage('regression Test no ZAP'){
		
			
			steps{
				
				sh "mvn -f ./RepoTwo/pom.xml clean -B test -DPETCLINIC_URL=${APP_URL}" 
				
				
			}
		}
		*/
		
		/*
		stage('regression Test with ZAP'){
		
			
			steps{
				sh '''
				APP_URL=http://${IP}:8888/petclinic
				ZAP_PORT="8443"
				ZAP_ENABLED='true'
				ZAP_IP=${IP}
				docker run --rm -v /zap/scripts/:/zap/scripts/:ro -p 8443:8443 -i owasp/zap2docker-stable zap.sh -daemon -config api.disablekey=true -config api.incerrordetails=true -config proxy.ip=0.0.0.0 -port 8443
				#mvn -f ./RepoTwo/pom.xml clean -B prepare-package -DPETCLINIC_URL=${APP_URL} -DZAP_ENABLED=${ZAP_ENABLED} -DZAP_IP=${ZAP_IP}
				./mvn clean -B test -DPETCLINIC_URL=${APP_URL} -DZAP_IP=${ZAP_IP} -DZAP_PORT=${ZAP_PORT} -DZAP_ENABLED=${ZAP_ENABLED}
				'''
			
            
			}
		}
		*/
		
		//In Progress...
		
		/*
		stage('performanceTestJob'){
		
			
			
			
				if [ ! -e apache-jmeter-2.13.tgz ]; then
            	wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-2.13.tgz
				fi
				tar -xf apache-jmeter-2.13.tgz
				
				
				ant -buildfile apache-jmeter-2.13/extras/build.xml -Dtestpath=${WORKSPACE}/RepoOne/src/test/jmeter -Dtest=petclinic_test_plan
				pwd
			
			steps{
				
				dir('RepoOne'){
				sh '''
				
				echo 'Changing user defined parameters for jmx file'
				sed -i 's/PETCLINIC_HOST_VALUE/'"52.16.226.150"'/g' ${WORKSPACE}/RepoOne/src/test/jmeter/petclinic_test_plan.jmx
				sed -i 's/PETCLINIC_PORT_VALUE/8888/g' ${WORKSPACE}/RepoOne/src/test/jmeter/petclinic_test_plan.jmx
				sed -i 's/CONTEXT_WEB_VALUE/petclinic/g' ${WORKSPACE}/RepoOne/src/test/jmeter/petclinic_test_plan.jmx
				sed -i 's/HTTPSampler.path"></HTTPSampler.path">petclinic</g' ${WORKSPACE}/RepoOne/src/test/jmeter/petclinic_test_plan.jmx		
				
				mvn verify 
				
				sed -i "s/###TOKEN_VALID_URL###/http:\\/\\/${IP}:8888/g" ${WORKSPACE}/RepoOne/src/test/gatling/src/test/scala/default/RecordedSimulation.scala 
				sed -i "s/###TOKEN_RESPONSE_TIME###/10000/g" ${WORKSPACE}/RepoOne/src/test/gatling/src/test/scala/default/RecordedSimulation.scala
				mvn -f src/test/gatling/ gatling:execute
				 '''
				 publishHTML(target: [
					reportDir : 'src/test/jmeter',
					reportFiles : 'petclinic_test_plan.html',
					reportName : 'J Meter Report',
					allowMissing : false,
					alwaysLinkToLastBuild : true,
					keepAll : true])

				gatlingArchive()
				}
					}
		}
		*/
		
		
		/*
		stage('deployProdA'){
		}
		stage('deployProdB'){
		}
		
		
  
*/

	}
}