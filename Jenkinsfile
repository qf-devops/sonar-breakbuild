node("maven-label") {
    def mvnHome
    stage('Preparation') { 
        
        git 'https://github.com/vytec-app/vytecapp.git'
        
        mvnHome = tool 'maven-3.6.3'
    }
     stage('code-quality') {
        
        withEnv(["MVN_HOME=$mvnHome"]) {
            if (isUnix()) {
                sh '"$MVN_HOME/bin/mvn" sonar:sonar -Dsonar.host.url=http://ip-172-31-11-16.us-west-2.compute.internal:9000'
            } else {
                bat(/"%MVN_HOME%\bin\mvn" -Dmaven.test.failure.ignore clean package/)
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

