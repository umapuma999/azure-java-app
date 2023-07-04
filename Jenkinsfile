import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=8ea4e627-3b1e-4c12-acb6-d130cc0eb905',
        'AZURE_TENANT_ID=be3c598a-5b62-40c3-8239-2e8adb92164e']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'uma1_group'
      def webAppName = 'test045'
      // login Azure
      withCredentials([usernamePassword(credentialsId: '<212f221c-f3eb-4813-aad4-56412074f90el>', passwordVariable: 'rEP8Q~t6Df0fTKPX2J06JaVN3Bn3Y~Npkqzexbv4', usernameVariable: '212f221c-f3eb-4813-aad4-56412074f90e')]) {
       sh '''
          az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_CLIENT_SECRET -t $AZURE_TENANT_ID
          az account set -s $AZURE_SUBSCRIPTION_ID
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
}
