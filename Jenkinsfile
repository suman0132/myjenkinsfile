pipeline {
  agent any
  environment {
        AWS_ACCOUNT_ID="896867441108"
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="aws-course-ecr"
        IMAGE_TAG="""$BUILD_ID"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"
    }
  stages{
    stage('Cloning Git') {
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/main']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[credentialsId: '', url: 'https://github.com/suman0132/myjenkinsfile.git']]])     
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
  }
}
