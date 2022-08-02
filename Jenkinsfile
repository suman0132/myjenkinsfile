pipeline {
  agent any
  evironment {
    registry = "account_id.dkr.ecr.us-east-2.amazonaws.com/my-docker-repo"
  }
  stages{
    stage('clean & compile') {
      steps{
        sh 'mvn clean compile'
      }
    }
    stage('unit-test') {
      steps {
        sh 'mvn test'
      }
      post {
        success {
          junit 'target/surefile-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }
    stage('package') {
      steps {
        sh 'mvn package -DskipTest'
      }
      post {
        success {
           echo 'Now Archiving...'
           archiveArtifacts artifacts: '**/target/*.war'
        }
      }
    }
    stage('integration-test') {
      steps {
        sh 'mvn verify -DskipUnitTests'
      }
    }
    stage('Docker Build') {
      steps {
        script {
          dockerimage = docker.build registry
        }
      }
    }
    stage('pushing to ECR') {
      steps{  
       script {
          sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin account_id.dkr.ecr.us-east-2.amazonaws.com'
          sh 'docker push account_id.dkr.ecr.us-east-2.amazonaws.com/my-docker-repo:latest'
       }
      }
    }
    stage('K8S Deploy') {
      steps{   
        script {
          withKubeConfig([credentialsId: 'K8S', serverUrl: '']) {
          sh ('kubectl apply -f  eks-deploy-k8s.yaml')
          }
        }
      }
    }
  }
}
