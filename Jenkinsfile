node
{
        
   properties([
        pipelineTriggers([
            pollSCM('*/2 * * * *')
        ])
    ])



def mvnhome= tool name: "Maven-practice"
echo "git branch Name: ${env.BRANCH_NAME}"
echo "build number: ${env.BUILD_NUMBER}"





   try
{
    stage ( 'code checkout from git')
    {
        git branch: 'DEV', url: 'https://github.com/kk-devops-notes/maven-webapplication-project-kkfunda.git'
    }
    stage ( 'complie code')
    {
       sh "${mvnhome}/bin/mvn compile"
    } 
      stage ( 'check code quality')
    {
       sh "${mvnhome}/bin/mvn  sonar:sonar"
    }   

     stage('SQ Approval') 
    {

        def decision = input(
            id: 'SQApproval',
            message: 'Please review the SonarQube report and approve/reject the pipeline.',
            parameters: [
                choice(
                    name: 'ACTION',
                    choices: ['Approve', 'Reject'],
                    description: 'Select your action'
                )
            ]
        )

        if (decision == 'Reject') 
        {
            error("Pipeline rejected after SonarQube review.")
        }

        echo "Pipeline approved. Proceeding to next stage..."
    }
    
      stage ( 'Build the artifact')
    {
       sh "${mvnhome}/bin/mvn  package"
    }  
      stage ( 'deploy to nexus')
    {
       sh "${mvnhome}/bin/mvn  deploy"
    }  
    
    stage('Deploy to Tomcat') 
    {
      
      sh """

      curl -u admin:123456 \
--upload-file /var/lib/jenkins/workspace/Test-SWPL/target/maven-web-application.war \
"http://3.110.147.27:8080/manager/text/deploy?path=/maven-web-application&update=true"
          
        """
    }

}

catch (e) {
   
       currentBuild.result = "FAILED"

  } finally {
    // Success or failure, always send notifications
    notifyBuild(currentBuild.result)       //function calling
  }

} //node ending

def notifyBuild(String buildStatus = 'STARTED') {
  // build status of null means successful
  buildStatus =  buildStatus ?: 'SUCCESS'

  // Default values
  def colorName = 'RED'
  def colorCode = '#FF0000'
  def subject = "${buildStatus}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
  def summary = "${subject} (${env.BUILD_URL})"

  // Override default values based on build status
  if (buildStatus == 'STARTED') {
    color = 'YELLOW'
    colorCode = '#FFFF00'
  } else if (buildStatus == 'SUCCESS') {
    color = 'GREEN'
    colorCode = '#27F584'
  } else {
    color = 'RED'
    colorCode = '#FF0000'
  }

  // Send notifications
  slackSend (color: colorCode, message: summary, channel: '#qa')


}
