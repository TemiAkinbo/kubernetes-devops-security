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

      stage('Mutation Tests - PIT') {
        steps {
          sh "mvn org.pitest:pitest-maven:mutationCoverage"
        }
        post {
          always {
            pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          }
        }
      }

      stage('SonarQube - SAST') {
      steps {
        withSonarQubeEnv('SonarQube'){ 
          sh "mvn sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.host.url=http://tadevsecopsdemo.eastus.cloudapp.azure.com:9000 -Dsonar.login=190402a112c4e037de34ba8d8fa096439ad7c9d6"
          }
          timeout(time: 2, units: 'MINUTES') {
            script {
              waitForQualityGate abortPipeline: true
            }
          }
        }
      }


      stage("Docker Build and Push"){
        steps {
          withDockerRegistry([credentialsId:"docker-hub", url:""]){
            sh 'printenv'
            sh 'docker build -t takinbo2410/numeric-app:""$GIT_COMMIT"" .'
            sh 'docker push takinbo2410/numeric-app:""$GIT_COMMIT""'
          }          
        }
      } 

      stage("Kube Deployment"){
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']){
            sh "sed -i 's#replace#takinbo2410/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
            sh "kubectl apply -f k8s_deployment_service.yaml"
          }          
        }
      } 
    }
}