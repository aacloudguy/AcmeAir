## Acmeair NodeJS on Bluemix 

Assume you have access to [Bluemix](https://console.ng.bluemix.net). 

	cf api api.ng.bluemix.net
	
	cf login

### Run Acmeair in Monolithic mode


#### Push Application

	cf push AA-OpsTraining-App-0001 --no-start -c "node app.js"

Note that "AA-OpsTraining-App-0001" is the application name and will be used for the hostname of the application. It needs to be unique so you need to add a unique identifier to it to make it unique. We use our four digit ID such as "0001"
		

#### Create any one of the services
	   
Mongo: 

	cf create-service compose-for-mongodb Standard AA-OpsTraining-DB-0001

   			

#### Bind service to application
	
	cf bind-service AA-OpsTraining-App-0001 AA-OpsTraining-DB-0001
	


#### Start application and Access application URL
	
	cf start AA-OpsTraining-App-0001
	
	http://AA-OpsTraining-App-0001.mybluemix.net	


### Run Acmeair in Microservices mode

#### You must first deploy the Acmeair in Monolithic mode before proceeding.

#### Push the authentication service

	cf push AA-OpsTraining-Auth-0001 --no-start -c "node authservice-app.js"

Again, "AA-OpsTraining-Auth-0001" needs to be unique.

#### Now that the authentication service is running, you can configure the web application to use it by setting the following user defined environment 

variable stopping the application first


	cf stop AA-OpsTraining-App-0001
	cf set-env AA-OpsTraining-App-0001 AUTH_SERVICE AA-OpsTraining-Auth-0001.mybluemix.net:80

#### Enable Hystrix by setting the following user defined environment variable

	cf set-env AA-OpsTraining-App-0001 enableHystrix true

#### Now start the authentication service and the web application


	cf start AA-OpsTraining-Auth-0001
	cf start AA-OpsTraining-App-0001

	Now go to http://AA-OpsTraining-App-0001.mybluemix.net and login. That login uses the authentication microservice.

#### You can run the Hystrix Dashboard in Bluemix as well. To deploy the Hystrix dashboard, you need to download the WAR file for the dashboard. You
can find a link to the download here: https://github.com/Netflix/Hystrix/wiki/Dashboard#installing-the-dashboard. The following CF CLI command will deploy 

the Hystrix dashboard to Bluemix:

	cf push AA-OpsTraining-Hystrix-0001 -p hystrix-dashboard-1.4.5.war

At the time of this writing, the latest version of the WAR file was 1.4.5. Make sure you get the latest version. Note that as before, "AA-OpsTraining-Hystrix-0001" needs to be unique. The WAR file will get deployed to the Liberty for Java runtime in Bluemix. Once the hystrix dashboard app is running, you will be able to access the dashboard using the following route:

	http://AA-OpsTraining-Hystrix-0001.mybluemix.net

To monitor the Acme Air authentication service, you need to monitor the following hystrix event stream:

	http://AA-OpsTraining-App-0001.mybluemix.net/rest/api/hystrix.stream

Specify that stream on the Hystrix Dashboard home page and click the Monitor.
