pipeline {
  agent any

  stages {
      stage('Build Artifact - Maven') {
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
              withSonarQubeEnv('SonarQube') {
                sh "mvn sonar:sonar \
                      -Dsonar.projectKey=numeric-application \
                      -Dsonar.host.url=http://devsecops-deland.eastus.cloudapp.azure.com:9000 \
                      -Dsonar.login=49dac8ec81ba66c33384dccf8137bd23ea400bfe"
              }
            //   timeout(time: 2, unit: 'MINUTES') {
            //     script {
            //       waitForQualityGate abortPipeline: true
            //     }
              
            // }
        }
      }
      // stage('Vulnerability Scan - Docker ') {
      //       steps {
      //           sh "mvn dependency-check:check"
      // }
      //       post {
      //             always {
      //                     dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
      //       }
      //       }
      // }
      stage('Vulnerability Scan - Docker') {
        steps {
          parallel(
            "Dependency Scan": {
              sh "mvn dependency-check:check"
            }, 
            "Trivy Scan": {
              sh "bash trivy-docker-image-scan.sh"
            }
          )
        }
      }
      stage('Docker Build and Push') {
        steps {
          withDockerRegistry([credentialsId: "docker-hub", url:""]) {
              sh 'printenv'
              sh 'sudo docker build -t awsdemo845/numeric-app .'
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

