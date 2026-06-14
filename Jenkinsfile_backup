node
{
        
   properties([
        pipelineTriggers([
            pollSCM('*/2 * * * *')
        ])
    ])

    
    def mvnhome = tool name: 'Maven-practice'
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

