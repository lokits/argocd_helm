node {
    def buildNumber = BUILD_NUMBER
    
      echo "GitHub BranhName ${env.BRANCH_NAME}"
      echo "Jenkins Job Number ${env.BUILD_NUMBER}"
      echo "Jenkins Node Name ${env.NODE_NAME}"
  
      echo "Jenkins Home ${env.JENKINS_HOME}"
      echo "Jenkins URL ${env.JENKINS_URL}"
      echo "JOB Name ${env.JOB_NAME}"
      
    stage('SCM Checkout'){
        git branch: 'main', url: 'https://github.com/arvindrg/argocd.git'
    }
    
    stage(" Maven Clean Package"){
      def mavenHome =  tool name: "maven-3.6.3", type: "maven"
      def mavenCMD = "${mavenHome}/bin/mvn"
      sh "${mavenCMD} clean package"
    } 
    
    stage("Build Dokcer Image") {
         sh "docker build -t gajjala7012/spring-boot-mongo:${buildNumber} ."
    }
    
    stage("Docker Push"){
        withCredentials([string(credentialsId: 'Docker_Hub_Pwd', variable: 'Docker_Hub_Pwd')]) {
          sh "docker login -u gajjala7012 -p ${Docker_Hub_Pwd}"
        }
        sh "docker push gajjala7012/spring-boot-mongo:${buildNumber}"
        
    }
    
    stage("Send EmailNotification") {
        emailext attachLog: true, body: '''"Pipeline Build is over .. Build # is ..${buildNumber} and Build status is.. ${currentBuild.result}."

Thanks 
Arvindh Reddy''', compressLog: true, replyTo: 'gajjalareddy5755@gmail.com', subject: '"Pipeline Build is over .. Build # is ..${buildNumber} and Build status is.. ${currentBuild.result}."', to: 'gajjalareddy5755@gmail.com'
}

}
