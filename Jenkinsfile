import groovy.json.JsonSlurper

def getFtpPublishProfile(def publishProfilesJson) {
  def pubProfiles = new JsonSlurper().parseText(publishProfilesJson)
  for (p in pubProfiles)
    if (p['publishMethod'] == 'FTP')
      return [url: p.publishUrl, username: p.userName, password: p.userPWD]
}

node {
  withEnv(['AZURE_SUBSCRIPTION_ID=c7a5dfe9-81bd-4d5a-b542-d15f774bc8d8',
        'AZURE_TENANT_ID=f32b97f0-efb8-4bc3-91ee-18a6e5f635c9']) {
    stage('init') {
      checkout scm
    }
  
    stage('build') {
      sh 'mvn clean package'
    }
  
    stage('deploy') {
      def resourceGroup = 'voya-openshift-jenkins-rg'
      def webAppName = 'voya-openshift-jenkins-webapp'
      // login Azure
      withCredentials([usernamePassword(credentialsId: 'azuresp', passwordVariable: 'e7wnCxJ3yeL3De4N.6KY7cn3PPlIU1.nrC', usernameVariable: '6bb7ac00-1234-4ef4-8bee-5bdd00171883')]) {
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
