node("maven-label") {
    def mvnHome
    stage('Preparation') { 
        
        git 'https://github.com/vytec-app/vytecapp.git'
        
        mvnHome = tool 'maven-3.6.3'
    }
     stage('code-quality') {
      withSonarQubeEnv('sonarqube-rec') {  
        withEnv(["MVN_HOME=$mvnHome"]) {
            
                sh '"$MVN_HOME/bin/mvn" sonar:sonar
         } 
        }
    }
    stage("SonarQube Quality Gate") { 
        timeout(time: 1, unit: 'HOURS') { 
           def qg = waitForQualityGate() 
           if (qg.status != 'OK') {
             error "Pipeline aborted due to quality gate failure: ${qg.status}"
           }
        }
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

