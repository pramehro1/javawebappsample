import groovy.json.JsonSlurper

def AZURE_CLIENT_ID = params.AZURE_CLIENT_ID = "none"
def AZURE_CLIENT_SECRET = params.AZURE_CLIENT_SECRET = "none"
def AZURE_TENANT_ID = params.AZURE_TENANT_ID = "none"

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
    bat 'mvn clean package'
  }
  
  stage('deploy') {
    def resourceGroup = 'PrateekResourceGroup' 
    def webAppName = 'DevopsForJava'
    // login Azure
    withCredentials([azureServicePrincipal('azsrvpricipal')]) {
      bat '''
        az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
        az account set -s $AZURE_SUBSCRIPTION_ID
      '''
    }
    // get publish settings
    def pubProfilesJson = bat script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
    def ftpProfile = getFtpPublishProfile pubProfilesJson
    // upload package
    bat "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
    // log out
    bat 'az logout'
  }
}
