pipeline {
  environment {
    registryName = "ovoh1"
    registry = "ovoh1/book-inventory-api"
    registryCredential = 'dockerhub'
    dockerImage = ''
  }
  agent any
  stages {
    stage('Cloning Git') {
      steps {
        git 'https://github.com/Egwonor/book-inventory-api.git'
      }
    }
    stage('Building jar') {
      steps {
       sh 'mvn clean install'
      }
    }
    stage('Building image') {
      steps{
        script {
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
    stage('Deploying image to Docker Hub') {
      steps{
        script {
          docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
    stage('Start Task on ECS Cluster') {
      steps{
        sh '''#!/bin/bash -x
          SERVICE_NAME="bookservice"
          IMAGE_VERSION="v_"${BUILD_NUMBER}
          TASK_FAMILY="booktask"
          
          # Create a new task definition for this build
          sed -e "s;%BUILD_NUMBER%;${BUILD_NUMBER};g" task-def-fargate.json > task-def-fargate-v_${BUILD_NUMBER}.json
          aws ecs register-task-definition --cli-input-json file://task-def-fargate-v_${BUILD_NUMBER}.json
          
          # Update the service with the new task definition and desired count
          TASK_REVISION=`aws ecs describe-task-definition --task-definition booktask | egrep "revision" | tr "/" " " | awk '{print $2}' | sed 's/"$//'`
          echo $TASK_REVISION
          DESIRED_COUNT="1"
          
          aws ecs update-service --cluster bookCluster --service ${SERVICE_NAME} --task-definition ${TASK_FAMILY} --desired-count ${DESIRED_COUNT}'''
      }
    }
    stage('Remove Unused docker image') {
      steps{
        sh "docker rmi $registry:$BUILD_NUMBER"
      }
    }
  }
}
