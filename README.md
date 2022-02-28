
# Demo of DevX with Tanzu 

This repo contains artifacts to run a demo illustrating the vision and capabilities of Tanzu for Dev, AppOps and Platform Ops

## Preperations 

- AKS and/or EKS access configured to support:
  - Loadbalancer 
  - Clustes creation permissions to match the specs in ```scripts/eks-handler.sh``` and ```scripts/aks-handler.sh```

- Rename the folder ```config-template``` to  ```.config``` 

- Fill blank crednetials in ```.config/tap-values.yaml```

- Update capatilized values in ```.config/tap-values.yaml```

  - ```$DOMAIN``` needs to be enabled to add wild-card DNS record to

- if planning to demo api-grid components 
  - Update ```workload/dekt4pets/dekt4pets-backend.yml```  to match ```tap-values```
  - update ```serverUrl:``` value in ```gateway/dekt4pets-gatway.yml``` and ```gateway/dekt4pets-gatway-dev.yml``` to match tap-values
  - Update ```host:``` value in ```workload/brownfield-apis``` files to match ```tap-values```

- The ingress setup is based on GoDaddy DNS, if you are using a different one, please modify ```scripts/ingress-handler.sh```

## Installation

### Core

- run ```./builder.sh init [aks / eks]```
  - install TAP full profile
  - install Spring Cloud Gateway (via HELM)
  - install the following Demo components 
    - App Accelerators
    - Source-to-api supply-chain
    - brownfield APIs (routes and gateways)
    - rabbitMQ operator and resource
  - setup dns and ingress rules 

### Add API-grid specific setup
- ```./api-grid.sh init```
- create the ```dekt4pets-backend``` and ```dekt4pets-frontend``` images
- setup SSO and app configs 
- deploy dekt4pets-dev-gateway

## Running the demo 

### Core 

- access tap gui accelerators via the ```cloud-native-devs``` tag
  - create ```devx-sensors``` workload using the boot-backend accelerator 
  - create ```devx-portal```  workload using the web-function accelerator 
  - use ```devx-mood-app```  as the parent application in both cases

- access the api-portal and highlight how discovery of existing APIs prior to creating new ones is done

- highlight the simplicity of the ```workload.yaml```

- show the simple tap installed command (don't actually run)
  - ```tanzu package install tap -p tap.tanzu.vmware.com -v 1.0.1  --values-file tap-values.yaml -n tap-install```

- show all the packages installed using ```tanzu package installed list -n tap-install```

- create workloads 
  - ```tanzu apps workload create -f workloads/devx-mood/mood-sensors.yaml -y```
  - ```tanzu apps workload create -f workloads/devx-mood/mood-portal.yaml -y```

- follow workload creation using ```tanzu apps workload list -n dekt-apps```

- access tap gui accelerators using the ```cloud-native-devsops``` tag
  - create ```source-to-api``` supplychain using the microservices-supplychain accelerator with ```web-backend``` workload type 
  - create ```source-to-api``` supplychain using the microservices-supplychain accelerator with ```web-frontend``` workload type

- highlight the separation of concerns between supplychain (AppOps) and supplychain-templates (Platform Ops)

- show applied supply chains using ```tanzu apps cluster-supply-chain list```
  - Note! The ```source-to-api``` supplychain is not active, the sensor is using the source-to-url as well 
 

- show supply chain and build service behind the scenes 
  - ```tanzu apps workload get mood-sensors -n dekt-apps```
  - ```tanzu apps workload tail mood-sensors --since 100m --timestamp  -n dekt-apps```

- access the live url of portal workload and show the call back to the sensors APIs 

- register a new entity in tap backstage gui
  -```https://github.com/dektlong/_DevXDemo/blob/main/workloads/devx-mood/backstage/catalog-info.yaml```
  - show system view diagram via ```devx-mood-app```
  - click down on ```mood-sensors``` to show application live view

- make a code change in ```mood-portal``` app to bypass the backend api calls 
  - https://github.com/dektlong/mood-portal/blob/main/main.go , change ALWAYS_HAPPY flag to true 
  - show how supply chain pickup the change and re run the path to prod
    - ```tanzu apps workload get mood-portal -n dekt-apps```
  - show a happy dog with sensors ignored

### API-grid specific example
- deploy dekt4pets-backend ```./api-grid.sh backend```   
  - show in api-portal dekt4pets-dev item added in real time  and how the front end team can discover and re-use backend APIs  

- deploy dekt4pets-frontend  ```./api-grid.sh frontend``` 
  - show in api portal how the frontend routes are added in real time 
- deploy a production  gateway with ingress access ```./api-grid.sh dekt4pets```
  - show in api-portal a new item dekt4pets
  - highlight the separation between routes and gateway runtime 
- Note! these apps are not using tap supply chain 

## Cleanup

- full cleanup to delete the cluster  ```./builder.sh cleanup [aks/eks]```

- partial cleanup to remove just workloads ```./builder.sh reset```

# Enjoy!



Extras

### Inner loop
- Access app accelerator developer instance  on ```acc.<APPS_APPS_SUB_DOMAIN>.<DOMAIN>```
- Development curated start 
  - Select ```onlinestore-dev``` tag
  - Select the ```Backend API for online-stores``` accelerator 
  - Select different deployment options and show generated files
  - Select different API-grid options and show generated files
- ```./demo.sh backend```
- Show how build service detects git-repo changes and auto re-build backend-image (if required)
- Show how the ```dekt4pets-gateway``` micro-gateway starts quickly as just a component of your app
- Access API Hub on ```api-portal.<APPS_APPS_SUB_DOMAIN>.<DOMAIN>```
  - Show the dekt4Pets API group auto-populated with the API spec you defined
  - now the frontend team can easily discover and test the backend APIs and reuse
  - Show the other API groups ('brownfield APIs')
- ```./demo.sh frontend```
- Access Spring Boot Observer at ```http://alv.<APPS_APPS_SUB_DOMAIN>.<DOMAIN>/apps``` to show actuator information on the backend application 
- Show the new frontend APIs that where auto-populated to the API portal

### Outer loop
- DevOps curated start 
  - Select ```onlinestore-devops``` tag
  - Select the ```API Driven Microservices workflow``` accelerator 
  - Select different deployment options and show generated files
  - Select different API-grid options and show generated files
  - Show the supply chain created via ```./demo.sh describe```
- ```./demo.sh dekt4pets```
  - show how the full supplychain for taking the app to production is manifested
- This phase will also add an ingress rule to the gateway, now you can show:
  - External traffic can only routed via the micro-gateway
  - Frontend and backend microservices still cannot be accessed directly) 
  - Access the application on 
  ```
  https://dekt4pets.<APPS_SUB_DOMAIN>.<DOMAIN>
  ```
  - login and show SSO functionality 

### Brownfield APIs
- now the backend team will leverage the 'brownfield' APIs to add background check functionality on potential adopters
- access the 'datacheck' API group and test adoption-history and background-check APIs
- explain that now other development teams can know exactly how to use a verified working version of both APIs (no tickets to off platform teams)

#### Demo brownfield API use via adding a route and patching the backend app
  - In ```workloads/dekt4pets/backend/routes/dekt4pets-backend-routes.yaml``` add
  ```
     - predicates:
        - Path=/api/check-adopter
        - Method=GET
      filters:
        - RateLimit=3,60s
      tags:
        - "Pets"    
  ```
  - In ```workloads/dekt4pets/backend/src/main/.../AnimalController.java``` add
  ```
	  @GetMapping("/check-adopter")
	public String checkAdopter(Principal adopter) {

		if (adopter == null) {
			return "Error: Invalid adopter ID";
		}

		String adopterID = adopter.getName();
    
		String adoptionHistoryCheckURI = "UPDATE_FROM_API_PORTAL" + adopterID;

   		RestTemplate restTemplate = new RestTemplate();
		
		  try
		  {
   			String result = restTemplate.getForObject(adoptionHistoryCheckURI, String.class);
		  }
		  catch (Exception e) {}

  		return "<h1>Congratulations,</h1>" + 
				"<h2>Adopter " + adopterID + ", you are cleared to adopt your next best friend.</h2>";
	}

  ```
  - ```./demo.sh backend -u ```
  - show how build-service is invoking a new image build based on the git-commit-id
  - run the new check-adopter api 
  ```
  dekt4pets.<APPS_SUB_DOMAIN>.<DOMAIN>/api/check-adopter
  ```
  - you should see the 'Congratulations...' message with the same token you received following login
#### Demo brownfield API use via a Cloud Native Runtime function
  - ```./demo.sh adopter-check ```
  - call the function via curl
  ```
    curl -w'\n' -H 'Content-Type: text/plain' adopter-check.dekt-apps.SERVING_SUB_DOMAIN.dekt.io \
    -d "datacheck.tanzu.dekt.io/adoption-history/109141744605375013560"
  ```
  - example output
  ```
    Running adoption history check..

    API: datacheck.tanzu.dekt.io/adoption-history/109141744605375013560
    Result: APPROVED

    Source: revision 1 of adopter-check
  ```
  - show how the function scales to zero after no use for 60 seconds
  ``` kubectl get pods -n dekt-apps ```
  - create a new revision
  ```./demo adopter-check -u ```
  - show how a new revision recieving 20% of the traffic is created

### Demo helper commands

tanzu package install tap -p tap.tanzu.vmware.com -v 1.0.1  --values-file tap-values.yaml -n tap-install

tanzu package installed list -n tap-install

tanzu apps workload create -f workloads/devx-mood/mood-sensors.yaml -y

tanzu apps workload create -f workloads/devx-mood/mood-portal.yaml -y

tanzu apps workload list -n dekt-apps

tanzu apps cluster-supply-chain list

tanzu apps workload get mood-sensors -n dekt-apps

tanzu apps workload tail mood-sensors --since 100m --timestamp  -n dekt-apps

https://github.com/dektlong/_DevXDemo/blob/main/workloads/devx-mood/backstage/catalog-info.yaml

