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
               def props = getProperties("target/sonar/report-task.txt")
               env.SONAR_CE_TASK_URL = props.getProperty('ceTaskUrl')
		      }
          }   
    } 
    stage("sonar-qualitygate"){
    withSonarQubeEnv('sonarqube-rec') {
        def ceTask
        timeout(time: 1, unit: 'MINUTES') {
          waitUntil {
            sh 'curl $SONAR_CE_TASK_URL -o ceTask.json'
            ceTask = jsonParse(readFile('ceTask.json'))
            echo ceTask.toString()
            return "SUCCESS".equals(ceTask["task"]["status"])
          }
        }
        def qualityGateUrl = env.SONAR_HOST_URL + "/api/qualitygates/project_status?analysisId=" + ceTask["task"]["analysisId"]
        sh "curl $qualityGateUrl -o qualityGate.json"
        def qualitygate = jsonParse(readFile('qualityGate.json'))
        echo qualitygate.toString()
        if ("ERROR".equals(qualitygate["projectStatus"]["status"])) {
          error  "Quality Gate failure"
        }
        echo  "Quality Gate success"
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

