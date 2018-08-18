import groovy.json.JsonSlurper

def AZURE_CLIENT_ID = params.AZURE_CLIENT_ID ?: "none"
def AZURE_CLIENT_SECRET = params.AZURE_CLIENT_SECRET ?: "none"
def AZURE_TENANT_ID = params.AZURE_TENANT_ID ?: "none"

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  stage('init') {
    checkout scm
  }
  
  stage('build') {
    sh 'mvn clean package'
  }
  
  stage('deploy') {
    def resourceGroup = 'PrateekResourceGroup' 
    def webAppName = 'DevopsForJava'
    // login Azure
    withCredentials([azureServicePrincipal('azsrvpricipal')]) {
      sh '''
        az login --service-principal -u dc01119c-2746-417e-b41c-3caf289cf397 -p Passw0rd@12345 -t d1e4f211-ccb8-4271-b93d-ecd282137fb9
        az account set -s c4bf10ac-3a10-401d-aa2a-612f33fd55cb
      '''
    }
    // get publish settings
    def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
    def ftpProfile = getFtpPublishProfile pubProfilesJson
    // upload package
    sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
    // log out
    sh 'az logout'
  }
}
