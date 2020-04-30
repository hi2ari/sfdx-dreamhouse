#!groovy
import groovy.json.JsonSlurperClassic
node {
	def BUILD_NUMBER=env.BUILD_NUMBER
    //def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
	def RUN_ARTIFACT_DIR="tests\\%BUILD_NUMBER%"
    //def SFDC_USERNAME="test-whnmqw0e9his@example.com"
	//def SFDC_USERNAME="test-73cjo4bujk8z@example.com"
	def SFDC_USERNAME
	def SFDC_TESTRUNID
	
	def HUB_ORG=env.SF_LOGINID
    def SFDC_HOST=env.SF_URL
    def JWT_KEY_CRED_ID=env.SERVER_KEY_CREDENTALS_ID
    def CONNECTED_APP_CONSUMER_KEY=env.SF_CONSUMER_KEY
	def toolbelt = tool 'toolbelt'
	
	println 'KEY IS' 
    println JWT_KEY_CRED_ID
	
	stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }
	withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')])
	{
		withEnv(["HOME=${env.WORKSPACE}"]) 
		{
				
			stage('Authorize DevHub') {
				if (isUnix()) {
					rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
				}else{
					rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${JWT_KEY_CRED_ID}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
				}
				if (rc != 0) { error 'hub org authorization failed' }
			}
			stage('Create Scratch Org'){
				 // need to pull out assigned username
				if (isUnix()) {
				  rmsg = sh returnStdout: true, script: "${toolbelt} force:org:create --definitionfile config/enterprise-scratch-def.json --json --setdefaultusername"
				}else{
					rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --setdefaultusername --setalias ciorg --wait 10 --durationdays 1"
				}
				printf rmsg
				println('Hello from a Job DSL script1!')
				println(rmsg)
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				println(beginIndex)
				println(endIndex)
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'org creation failed: ' + robj.message }
				SFDC_USERNAME=robj.result.username
				robj = null
			}
			stage('Push To Test Org') {
				if (isUnix()) {
					rc = sh returnStatus: true, script: "${toolbelt} force:source:push --json --targetusername ${SFDC_USERNAME}"
				}else{
					//rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --json --targetusername ${SFDC_USERNAME}"
					rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:push --json --targetusername ${SFDC_USERNAME}"
					//rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:source:push --json --targetusername ciorg"
				}
				printf rmsg
				println('Hello from a Job DSL script2!')
				println(rmsg)
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				println(beginIndex)
				println(endIndex)
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'push failed: ' + robj.message }
				robj = null
			}
			stage('Assign Permset') {
				//assign permset
				if (isUnix()) {
					rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
				}else{
					//rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
					rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:user:permset:assign --json --targetusername ${SFDC_USERNAME} --permsetname sfdx-jenkins-ci"
				}
				printf rmsg
				println('Hello from a Job DSL script3!')
				println(rmsg)
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				println(beginIndex)
				println(endIndex)
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'permset assignment failed: ' + robj.message }
				robj = null
			}
			stage ('Import Data') {
				if (isUnix()) {
					rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:data:tree:import --plan data/sample-data-plan.json --json"
				}else{
					//rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname sfdx-jenkins-ci"
					rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:data:tree:import --plan data/sample-data-plan.json --json"
				}
				printf rmsg
				println('Hello from a Job DSL script3.5!')
				println(rmsg)
				def beginIndex = rmsg.indexOf('{')
				def endIndex = rmsg.indexOf('}')
				println(beginIndex)
				println(endIndex)
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'Data import failed: ' + robj.message }
				//SFDC_TESTRUNID = robj.result.summary.testRunId
				robj = null
			}
		stage ('Run Apex Tests') {
				if (isUnix()){
					sh "mkdir -p ${RUN_ARTIFACT_DIR}"
					timeout(time: 120, unit: 'SECONDS') {
					rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
					}
				}else{
					bat "mkdir ${RUN_ARTIFACT_DIR}"
					timeout(time: 120, unit: 'SECONDS') {
					//rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
					rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --json --targetusername ${SFDC_USERNAME}"
					}
				}
				println('Hello from a Job DSL script4!')
				println(rmsg)
				def beginIndex = rmsg.indexOf('{')
				println(beginIndex)
				def endIndex = rmsg.indexOf('}')
				println(endIndex)
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'Apex test run failed: ' + robj.message }
				SFDC_TESTRUNID = robj.result.summary.testRunId
				println(robj.result.summary.testRunId)
				println SFDC_TESTRUNID
				robj=null
				
			}
		stage ('Apex Test Report') {
			if (isUnix()){
					//rc = sh returnStatus: true, script: "${toolbelt}/sfdx force:apex:test:report -i ${SFDC_TESTRUNID} --resultformat human --json
				}else{
					//rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:report -i ${SFDC_TESTRUNID} --resultformat human --json"
					rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:apex:test:report -i ${SFDC_TESTRUNID} --resultformat human --json"
				}
				println('Hello from a Job DSL script5!')
				println(rmsg)
				def beginIndex = rmsg.indexOf('{')
				println(beginIndex)
				def endIndex = rmsg.indexOf('}')
				println(endIndex)
				def jsobSubstring = rmsg.substring(beginIndex)
				println(jsobSubstring)
				def jsonSlurper = new JsonSlurperClassic()
				def robj = jsonSlurper.parseText(jsobSubstring)
				if (robj.status != 0) { error 'Apex test run failed: ' + robj.message }
				robj=null
			}
			
			stage('collect results') {
				junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
			}
		}
	}
}
