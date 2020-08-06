//test
node("maven-label") {
    def mvnHome
    stage('Preparation') { 
        
        git 'https://github.com/vytec-app/vytecapp.git'
        
        mvnHome = tool 'maven-3.6.3'
    }
     
    stage("sonar"){
      withSonarQubeEnv('sonarqube-rec') {
              withEnv(["MVN_HOME=$mvnHome"]) {      
               
               sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.2:sonar'
            
		      }
          }   
    } 
    stage("sonar-qualitygate"){
    sh './breakbuild.sh http://ip-172-31-11-16.us-west-2.compute.internal:9000/'
    }
    stage('Build') {
        
        withEnv(["MVN_HOME=$mvnHome"]) {
             withSonarQubeEnv('SonarQube') {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" -Dmaven.test.failure.ignore -P vytecapp-deploy clean deploy'
            } else {
                bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
            }
             }
        }
    }
    stage('Results') {
        junit '**/target/surefire-reports/TEST-*.xml'
        archiveArtifacts 'target/*.jar'
    }
}

