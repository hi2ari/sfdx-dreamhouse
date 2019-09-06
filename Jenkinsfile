#!groovy
import groovy.json.JsonSlurperClassic
node {
	def BUILD_NUMBER=env.BUILD_NUMBER
    //def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
	def RUN_ARTIFACT_DIR="tests\\%BUILD_NUMBER%"
    //def SFDC_USERNAME="test-whnmqw0e9his@example.com"
	def SFDC_USERNAME="test-73cjo4bujk8z@example.com"
	def SFDC_TESTRUNID
	
	def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST=env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID=env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
	def toolbelt = tool 'toolbelt'
	
	println 'KEY IS' 
    println JWT_KEY_CRED_ID
	
	stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }
	//withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')])
	//{
		stage('Authorize DevHub') {
			if (isUnix()) {
				rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
			}else{
				rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${JWT_KEY_CRED_ID}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
			}
			if (rc != 0) { error 'hub org authorization failed' }
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
			}
		
		stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
        }
	//}
}
