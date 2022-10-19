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
					//delete the old package content so that only the latest content gets stored
					//dir(env.GITFolder + '/' + env.IntegrationPkg){
					//	deleteDir();
					//}
					//download and extract package from tenant
					println("Downloading package");
					def tempfile = UUID.randomUUID().toString() + ".zip";
					//def tempfile = node.'*:properties'.'*:Id'.text() + ".zip";
					//println("here is the random value:" + tempfile);
					
					
					def cpiDownloadResponse1 = httpRequest httpProxy: 'http://rb-proxy-sl.rbesz01.com:8080', 
						customHeaders: [[maskValue: false, name: 'Authorization', value: token, name: 'Content-Type', value: 'atom/xml']], 
						ignoreSslErrors: false, 
						responseHandle: 'LEAVE_OPEN', 
						//validResponseCodes: '100:399, 404',
						validResponseCodes: '200:404',
						timeout: 30,  
						outputFile: tempfile,
					url: 'https://' + env.CPIHost + '/api/v1/IntegrationPackages';
					println("here is the information of URL:"+ tempfile);
					
					if (cpiDownloadResponse1.status == 404){
						//invalid Package ID
						error("Received http status code 404. Please check if the Package ID that you have provided exists on the tenant.");
					}
					//def disposition = cpiDownloadResponse1.headers.toString();
					//def index=disposition.indexOf('filename')+9;
					//def lastindex=disposition.indexOf('.zip', index);
					//def filename=disposition.substring(index + 1, lastindex + 4);
					//def folder=env.GITFolder + '/' + filename.substring(0, filename.indexOf('.zip'));
					//println("Before fileOperation")
					//fileOperations([fileUnZipOperation(filePath: tempfile, targetLocation: folder)])
					cpiDownloadResponse1.close();
					//println("After fileOperation")
					//remove the zip
					//fileOperations([fileDeleteOperation(excludes: '', includes: tempfile)])
					//println("After fileDeleteOperation")	
					dir(folder){
						//println("Now in the git add command place")
						sh 'git add .'
						//println("After the git add command")
					}
					//println("Store integration package in Git")
					//withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: env.GITCredentials ,usernameVariable: 'GIT_AUTHOR_NAME', passwordVariable: 'GIT_PASSWORD']]) {  
						//sh 'git diff-index --quiet HEAD || git commit -am ' + '\'' + env.GitComment + '\''
						//sh('git push --push-option=ci-skip https://${GIT_AUTHOR_NAME}:${GIT_PASSWORD}@' + env.GITRepositoryURL + ' HEAD:' + env.GITBranch)
					//}
					
				}
			}
		}
    }
}
