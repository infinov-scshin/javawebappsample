import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=b8d5d579-b582-4fa9-801f-adf4ab3eabe6',
        'AZURE_TENANT_ID=489a657d-fb55-42eb-9963-010a9e15264e']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      bat 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'scshintest'
      def webAppName = 'scshintestapp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'AzureServicePrincipal', passwordVariable: '_uyMzaLOpq5K2Or3p1j_A_V19Fi8z_yBto', usernameVariable: 'e989cd22-c96a-4632-847f-ec4a12026e56')]) {
       bat '''
          az login --service-principal -u e989cd22-c96a-4632-847f-ec4a12026e56 -p _uyMzaLOpq5K2Or3p1j_A_V19Fi8z_yBto -t 489a657d-fb55-42eb-9963-010a9e15264e
          az account set -s b8d5d579-b582-4fa9-801f-adf4ab3eabe6
        '''
      }
      // get publish settings
      def pubProfilesJson = sh script: "az webapp deployment list-publishing-profiles -g $resourceGroup -n $webAppName", returnStdout: true
      def ftpProfile = getFtpPublishProfile pubProfilesJson
      // upload package
      bat "curl -T target/calculator-1.0.war $ftpProfile.url/webapps/ROOT.war -u '$ftpProfile.username:$ftpProfile.password'"
      // log out
      bat 'az logout'
    }
  }
}
