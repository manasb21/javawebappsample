import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}
 node {
  withEnv(['AZURE_SUBSCRIPTION_ID=80a14f2f-5eff-4f5b-a9f5-9ca0878341fb',
        'AZURE_TENANT_ID=5855e6b5-0c59-47e0-bb93-ae4049a67315']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
        def mvn_version = 'defaultmaven'
        withEnv(["PATH+MAVEN=${tool mvn_version}/bin"] ) {
          sh 'mvn clean package'
        }
    }
  
    stage('deploy') {
      
        
      def resourceGroup = 'spoke-dev-vnet-rg'
      def webAppName = 'sampleapp'
      // login Azure

       withCredentials([usernamePassword(credentialsId: 'spoke-dev-vnet-rg-owner', passwordVariable: 'AZURE_CLIENT_SECRET', usernameVariable: 'AZURE_CLIENT_ID')]) {
        //sh '''
               
           //az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
           //az account set -s $AZURE_SUBSCRIPTION_ID
        //'''
       azureCLI commands: [[exportVariablesString: '', script: 'az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID'], [exportVariablesString: '', script: 'az account set -s $AZURE_SUBSCRIPTION_ID']], principalCredentialId: 'spoke-dev-vnet-rg-owner'
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      azureCLI commands: [[exportVariablesString: '', script: 'curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u "$ftpProfile.username:$ftpProfile.password"']], principalCredentialId: 'spoke-dev-vnet-rg-owner'
      //sh "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
     azureCLI commands: [[exportVariablesString: '', script: 'az logout']], principalCredentialId: 'spoke-dev-vnet-rg-owner'
      //sh 'az logout'
      
    }
  }
}
             
