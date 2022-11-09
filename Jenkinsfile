pipeline {
	agent any

	//Configure the following environment variables before executing the Jenkins Job	
	environment {
		//IntegrationPkg = "muthuPOC"
		CPIHost = "${env.CPI_HOST}"
		CPIOAuthHost = "${env.CPI_OAUTH_HOST}"
		CPIOAuthCredentials = "${env.CPI_OAUTH_CRED}"	
		GITRepositoryURL  = "${env.GIT_REPOSITORY_URL}"
		GITCredentials = "${env.GIT_CRED}"
		GITBranch = "${env.GIT_BRANCH_NAME}"
		GITFolder = "IntegrationPackages"
		GITComment = "Integration Package update from CICD pipeline"
  	}
	
	stages {
		stage('download integration package and store it in Git') {
			steps {
			 	deleteDir()
				script {
					//clone repo 
					checkout([
						$class: 'GitSCM',
						branches: [[name: env.GITBranch]],
						doGenerateSubmoduleConfigurations: false,
						extensions: [
							[$class: 'RelativeTargetDirectory',relativeTargetDir: "."],
							[$class: 'SparseCheckoutPaths',  sparseCheckoutPaths:[[$class:'SparseCheckoutPath', path: env.GITFolder]]]
						],
						submoduleCfg: [],
						userRemoteConfigs: [[
							credentialsId: env.GITCredentials,
							url: 'https://' + env.GITRepositoryURL
						]]
					])
					//get token
					println("Request token");
					def token;
					def tempfile = node.'*:properties'.'*:Id'.text() + ".zip";
					try{
						def getTokenResp = httpRequest httpProxy: 'http://rb-proxy-sl.rbesz01.com:8080',acceptType: 'APPLICATION_JSON', 
						authentication: env.CPIOAuthCredentials, 
						//edit
						contentType: 'APPLICATION_JSON', 
						httpMode: 'POST', 
						responseHandle: 'LEAVE_OPEN', 
						timeout: 30, 
					url: 'https://' + env.CPIOAuthHost + '/oauth/token?grant_type=client_credentials';
					def jsonObjToken = readJSON text: getTokenResp.content
					token = "Bearer " + jsonObjToken.access_token
				   	} catch (Exception e) {
						error("Requesting the oauth token for Cloud Integration failed:\n${e}")
					}
					//def tempfile = UUID.randomUUID().toString();
					//def tempfile = node.'*:properties'.'*:Id'.text() + ".zip";
					println("here is the random value:" + tempfile);
									
					def cpiDownloadResponse1 = httpRequest httpProxy: 'http://rb-proxy-sl.rbesz01.com:8080', 
						customHeaders: [[maskValue: false, name: 'Authorization', value: token, ContentType: 'application/atom+xml;type=feed;charset=utf-8' ]], 
						ignoreSslErrors: false, 
						httpMode: 'GET', 
						responseHandle: 'LEAVE_OPEN', 
						validResponseCodes: '100:399, 404',
						//validResponseCodes: '200:404',
						timeout: 30,  
						outputFile: tempfile,
					url: 'https://' + env.CPIHost + '/api/v1/IntegrationPackages';
					println("here is the information of URL:"+ tempfile);
					
					if (cpiDownloadResponse1.status == 404){
						//invalid Package ID
						error("Received http status code 404. Please check if the Package ID that you have provided exists on the tenant.");
					}
					def disposition1 = cpiDownloadResponse1.headers.toString();
					def index1=disposition1.indexOf('filename1')+9;
					def lastindex1=disposition1.indexOf('.zip', index1);
					def filename1=disposition1.substring(index1 + 1, lastindex1 + 4);
					def folder1=env.GITFolder + '/' + filename1.substring(0, filename1.indexOf('.zip'));
					//println("Before fileOperation")
					fileOperations([fileUnZipOperation(filePath: tempfile, targetLocation: folder1)])
					cpiDownloadResponse1.close();
					
					//def body = tempfile;
        				//def feed = new XmlParser().parseText(body);
       					 //checks  
        				//assert feed.entry instanceof groovy.util.NodeList 
       					//assert feed.title.text() == 'IntegrationPackages' 
      
       					//Iterator itt = feed.entry.iterator();
        				//List result = new ArrayList();
        				//while (itt.hasNext()) {
          				//Node node = (Node) itt.next();
          				//result.add(node.'*:properties'.'*:Id'.text());
          				//println(node.'*:properties'.'*:Id'.text());
       					// }

					//println("Downloading package");
					//def tempfile = UUID.randomUUID().toString() + ".zip";
					//def tempfile = IntegrationPkg + ".zip";
					//println("here is the random value:" + tempfile);
					//def cpiDownloadResponse = httpRequest httpProxy: 'http://rb-proxy-sl.rbesz01.com:8080',acceptType: 'APPLICATION_ZIP', 
					//	customHeaders: [[maskValue: false, name: 'Authorization', value: token]], 
					//	ignoreSslErrors: false, 
					//	responseHandle: 'LEAVE_OPEN', 
					//	validResponseCodes: '100:399, 404',
					//	timeout: 30,  
					//	outputFile: tempfile,
						//url: 'https://' + env.CPIHost + '/api/v1/IntegrationPackages(\''+ env.IntegrationPkg + '\')/$value';
					//	url: 'https://' + env.CPIHost + '/api/v1/IntegrationPackages(\''+ node.'*:properties'.'*:Id'.text() + '\')/$value';
					//if (cpiDownloadResponse.status == 404){
						//invalid Package ID
					//	error("Received http status code 404. Please check if the Package ID that you have provided exists on the tenant.");
					//}
					//def disposition = cpiDownloadResponse.headers.toString();
					//def index=disposition.indexOf('filename')+9;
					//def lastindex=disposition.indexOf('.zip', index);
					//def filename=disposition.substring(index + 1, lastindex + 4);
					//def folder=env.GITFolder + '/' + filename.substring(0, filename.indexOf('.zip'));
					//println("Before fileOperation")
					//fileOperations([fileUnZipOperation(filePath: tempfile, targetLocation: folder)])
					//cpiDownloadResponse.close();
					
					
					//println("After fileOperation")
					//remove the zip
					//fileOperations([fileDeleteOperation(excludes: '', includes: tempfile)])
					//println("After fileDeleteOperation")	
					dir(folder){
						sh 'git add .'
						}
					println("Store integration package in Git")
					//withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: env.GITCredentials ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {  
						//sh 'git diff-index --quiet HEAD || git commit -am ' + '\'' + env.GitComment + '\''
						//sh('git push --push-option=ci-skip https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@' + env.GITRepositoryURL + ' HEAD:' + env.GITBranch)
					//}
					
				}
			}
		}
    }
}
