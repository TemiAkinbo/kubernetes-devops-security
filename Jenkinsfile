pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }   

      stage('Unit Tests - JUnit and JaCoCo') {
            steps {
              sh "mvn test"
            }
             post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }

      stage("Docker Build"){
        steps {
          sh 'printenv'
          sh 'docker build -t takinbo2410/numeric-app:""$GIT_COMMIT""'
          sh 'docker push takinbo2410/numeric-app:""$GIT_COMMIT""'
        }
      } 
    }
}