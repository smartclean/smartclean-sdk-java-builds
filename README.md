# smartclean-sdk-java

## Introduction
This is a fully integrated java SDK to facilitate a client with access to all the features and reporting tools for various modules hosted over the SmartClean console.

## Modules
The SDK consists of the following modules:
- wfm (Work-Force Management)
- matrix (Matrix)
- optimus (Optimus)
- scteamsprop (Smartclean Teams & Properties)
- auth (Authentication & Authorization)
- globalUtils (SDK Global Utils)
- httpClient (Http Client)

> Please refer to README.md of respective modules to get detailed info.

## Dependency Structure
The following image best describes the dependency structure of the SDK as a whole and the inter-dependency among the different modules.

![archDiagram](https://github.com/smartclean/smartclean-sdk-java/blob/master/SC-SDK-dependency-diagram.drawio-2.png)

## Steps to integrate
Following steps can be followed to integrate the SDK to a maven project:
- download the sdk jar file from the releases section of this repository.
- open command window on your system and run the following command:-
> replace <path-to-jar-file> with the path of the jar file in your system, replace <group-id> with 'org.smartclean', replace <artifact-id> with 'smartclean-java-sdk', replace <version  with '1b' & replace <packaging with 'jar'.
```
mvn install:install-file \
   -Dfile=<path-to-jar-file> \
   -DgroupId=<group-id> \
   -DartifactId=<artifact-id> \
   -Dversion=<version> \
   -Dpackaging=<packaging> \
   -DgeneratePom=true
```

> running the above command should install the jar in your local .m2 repository and you should be able to declare the jar as dependency in your project pom.

- add below dependency in your project pom

```
    <dependency>
        <groupId>org.smartclean</groupId>
        <artifactId>smartclean-java-sdk</artifactId>
        <version>1b</version>
    </dependency>
```
- place sdk-config.yml & sc-tenants.yml in the root directory of the project. The directory structure of the project should look like this:-

![archDiagram](https://github.com/smartclean/smartclean-sdk-java/blob/master/project-directory-tree.png)

> **Note:** Contents of the target directory in the above image depends on the local build setup of the client project.

- create the instance of interface `SCSDKClientInterface`.

## Configuring SDK
SDK can be configured through environment variables or through the config file `sdk-config.yml`. SDK can be configured to run in two different modes 'production' OR 'local'. SDK will at first look for the environment variables, if the required environment variables are not found it will, by default, try to initialize in 'production' mode for which it will look for `sdk-config.yml` in the project base directory. `sdk-config.yml` is not required if `MODE`
is set as `local` in the environment variables. Following environment variables are required to be set to run the sdk in local mode:

```
    MODE, //production OR local; default production
    MODULE_URL_HOST, //default https://www.smartclean.io/matrix/utils/modules/moduleversions.json
    STALE_CACHE_THRESHOLD, //default 900
    CACHE_CLEANUP_FREQUENCY, //default 600
    PROTOCOL, // http OR https
    SC_WFM_HOST,
    SC_WFM_PORT,
    SC_WFM_VERSION,
    SC_DM_HOST,
    SC_DM_PORT,
    SC_DM_VERSION,
    SC_GRIDS_HOST,
    SC_GRIDS_PORT,
    SC_GRIDS_VERSION,
    SC_METRICS_HOST,
    SC_METRICS_PORT,
    SC_METRICS_VERSION,
    SC_TEAMS_PROP_HOST,
    SC_TEAMS_PROP_PORT,
    SC_TEAMS_PROP_VERSION,
    ENABLE_SIGNING; // true OR false; default true
```

## Example
Following code snippets shows how to use the sdk in code once the sdk has been integrated using the steps described above:
- In a java web-app

```
public class ScsdkClientAppApplication {

        // main sdk interface that allows access to the implemented operations under different modules
	private static SCSDKClientInterface clientInterface;
	
	    // object mapper to log object response from the sdk as json string
	static ObjectMapper objectMapper = new ObjectMapper();
	
	public static void main(String[] args) throws JsonProcessingException {
		clientInterface = new SCSDKMasterService();
		
		// WFMMainRequestObject is the master object to structure the request body required for making api calls to the operations provided under the workforcemanagement module
		WFMMainRequestObject mrq = RequestObject.WFM.getShiftDetailsRequestObject("06f98e0c4f464b84b9157b1df6adfa66_003","a0801fe3-40f7-4127-abc0-fa6d58cbcebf");
		
		// WFMSDKRequestObject is the main request object to structure the api params, headers, request-body for the wfm module operations
		WFMSDKRequestObject requestObject = new WFMSDKRequestObject("","181018be1bd74d349e04bbd62ff2e6a4",
				"06f98e0c4f464b84b9157b1df6adfa66","SMARTCLEAN", mrq, "scworkforcemanagement");
		try{
		        // WFMMasterResponse is the master response object of the WFM module.
			WFMMasterResponse shiftDR = clientInterface.getShiftDetailsByID(requestObject);
			System.out.println(objectMapper.writer().withDefaultPrettyPrinter().writeValueAsString(shiftDR));
            if(shiftDR.getHttpResponseStatus() == 202) {
                ShiftDetailsByID shiftDetailsByID = objectMapper.convertValue(shiftDR.getData(), ShiftDetailsByID.class);
                System.out.println("data ===> "+objectMapper.writer().withDefaultPrettyPrinter().writeValueAsString(shiftDetailsByID));
            }
		}catch (Exception exp){
			exp.printStackTrace();
		}finally {
		        // system exit is called to shutdown the sdk cache handler. 
			System.exit(0);
		}
	}
}
```

- In a SpringBoot Web Application

```
// main application class

@SpringBootApplication
public class ScsdkClientAppApplication {

        // main sdk interface that allows access to the implemented operations under different modules
        @Autowired
	private SCSDKClientInterface clientInterface;
	
	private ObjectMapper objectMapper = new ObjectMapper();
	
	public static void main(String[] args) throws JsonProcessingException {
		SpringApplication.run(ScsdkClientAppApplication.class, args);
	}
	
	// method auto-runs after dependency-injection to ensure the autowired fields have been made available to this class
	@PostConstruct
	private void runAfterDI() {
	
	        // WFMMainRequestObject is the master object to structure the request body required for making api calls to the operations provided under the workforcemanagement module
		WFMMainRequestObject mrq = RequestObject.WFM.getShiftDetailsRequestObject("06f98e0c4f464b84b9157b1df6adfa66_003","a0801fe3-40f7-4127-abc0-fa6d58cbcebf");
		
		// WFMSDKRequestObject is the main request object to structure the api params, headers, request-body for the wfm module operations
		WFMSDKRequestObject requestObject = new WFMSDKRequestObject("","181018be1bd74d349e04bbd62ff2e6a4",
				"06f98e0c4f464b84b9157b1df6adfa66","SMARTCLEAN", mrq, "scworkforcemanagement");
		try{
		        // WFMMasterResponse is the master response object of the WFM module.
			WFMMasterResponse shiftDR = clientInterface.getShiftDetailsByID(requestObject);
			System.out.println(objectMapper.writer().withDefaultPrettyPrinter().writeValueAsString(shiftDR));
            if(shiftDR.getHttpResponseStatus() == 202) {
                ShiftDetailsByID shiftDetailsByID = objectMapper.convertValue(shiftDR.getData(), ShiftDetailsByID.class);
                System.out.println("data ===> "+objectMapper.writer().withDefaultPrettyPrinter().writeValueAsString(shiftDetailsByID));
            }
		}catch (Exception exp){
			exp.printStackTrace();
		}finally {
		        // system exit is called to shutdown the sdk cache handler. 
			System.exit(0);
		}
	}

}


// application configuration class

@Configuration
public class ClientApplicationConfiguration {

    // Main sdk interface defined as bean to be autowired in the application whereever required
    @Bean
    public SCSDKClientInterface scSDKClientInterface() {
        return new SCSDKMasterService();
    }
}
```

> Refer to the `SCSDKClientApp` folder to get the example project that implements the SDK.

## Bug reporting and Feature requests
Please create an issue in this github repository with the necessary details.
>Please refer to issue_templates folder to check out the formats for bug & feature reports.
