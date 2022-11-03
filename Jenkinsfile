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
      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url:""]) {
              sh 'printenv'
              sh 'docker build -t awsdemo845/numeric-app .'
              sh 'docker push awsdemo845/numeric-app'
          }   
        }
      }
      stage ('Kubernetes Deployment - DEV') {
        steps {
          withKubeConfig([credentialsId: 'kubeconfig']){
            // sh "sed -i 's#replace#awsdemo845/numeric-app#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
          }
          
        }
      }
    }
}

