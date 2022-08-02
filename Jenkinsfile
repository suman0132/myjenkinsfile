pipeline {
  agent any
  environment {
        AWS_ACCOUNT_ID="YOUR_ACCOUNT_ID_HERE"
        AWS_DEFAULT_REGION="CREATED_AWS_ECR_CONTAINER_REPO_REGION" 
        IMAGE_REPO_NAME="ECR_REPO_NAME"
        IMAGE_TAG=env.BUILD_NUMBER
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
  stages{
    stage('Cloning Git') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/akannan1087/springboot-app']]])     
      }
    }
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
          dockerimage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
    stage('pushing to ECR') {
      steps{  
       script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
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
