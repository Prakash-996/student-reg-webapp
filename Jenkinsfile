node {
    try{
    def maven_home = tool name: 'Maven3.9.12', type: 'maven'
    def tomcat_serverid="172.31.43.193"
   // stage("git clone"){
   //     git branch: 'development', credentialsId: 'git_hub_credentials', url: 'https://github.com/Prakash-996/student-reg-webapp.git'
   // }
    stage("maven package"){
        sh "${maven_home}/bin/mvn clean package"
    }
    stage("sonarscan"){
        withCredentials([string(credentialsId: 'sonarqube_token', variable: 'sonartoken')]){
            sh "${maven_home}/bin/mvn clean verify sonar:sonar -Dsonar.token=${sonartoken}"
        }
    }
    stage("upload war file to nexus"){
        sh "${maven_home}/bin/mvn clean deploy"
    }
    stage("upload war file to tomcat"){
        sshagent(['tomcat_ssh_cred']) {
              sh"""
            ssh -o StrictHostKeyChecking=no ec2-user@${tomcat_serverid} sudo systemctl stop tomcat
            sleep 20
            ssh -o StrictHostKeyChecking=no ec2-user@${tomcat_serverid} rm /opt/tomcat/webapps/student-reg-webapp.war
            scp -o StrictHostKeyChecking=no target/student-reg-webapp.war ec2-user@${tomcat_serverid}:/opt/tomcat/webapps/student-reg-webapp.war
            ssh -o StrictHostKeyChecking=no ec2-user@${tomcat_serverid} sudo systemctl start tomcat
            """
}
   }
    }catch(Exception e){
        currentBuild.result ='FAILURE'
    }finally{
        def buildstatus = currentBuild.result ?: 'SUCCESS'
        def colorcode = 'good'
        
        if(buildstatus =='FAILURE'){
            colorcode='danger'
        }
        //slackSend channel: '#all-rushitech' , color: "${colorcode}" , message :"Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER} - ${env.buildstatus} - Please check the output ${env.BUILD_URL}"
        emailext body:"Jenkins Job ${env.JOB_NAME} - ${env.BUILD_NUMBER} - ${env.buildstatus} - Please check the output ${env.BUILD_URL}", subject: '${env.JOB_NAME} - ${env.BUILD_NUMBER} ', to: 'chennuruprakash19@gmail.com'
        }
}