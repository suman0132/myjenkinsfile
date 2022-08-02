pipeline {
  agent any
  environment {
         registry = "896867441108.dkr.ecr.ap-south-1.amazonaws.com/aws-course-ecr"
    }
  stages{
    stage('Cloning Git') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/lerndevops/samplejavaapp.git']]])     
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
          dockerimage = docker.build registry 
        }
      }
    }
    stage('pushing to ECR') {
      steps{  
       script {
                sh 'aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 896867441108.dkr.ecr.ap-south-1.amazonaws.com'
                sh 'docker push 896867441108.dkr.ecr.ap-south-1.amazonaws.com/aws-course-ecr:env.BUILD_NUMBER'
      }
    }
  }
}
