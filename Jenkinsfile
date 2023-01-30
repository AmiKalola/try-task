pipeline{
    agent any
   
    stages {
        stage('docker ecr login') {
            steps {
                sh "echo login to aws ecr"
                sh 'aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 017690734226.dkr.ecr.us-east-1.amazonaws.com'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "echo staring build the image"
                sh 'docker build -t ecs-demo-2 .'
            }
        }
        stage('Deploy Docker Image') {
            steps {
                sh "echo staring deploying the image"
                sh 'docker tag ecs-demo-2:latest 017690734226.dkr.ecr.us-east-1.amazonaws.com/ecs-demo-2:latest'
                sh 'docker push 017690734226.dkr.ecr.us-east-1.amazonaws.com/ecs-demo-2:latest'
            }
        }      
        stage('update cluster') {
            steps {
              sh '''
              NEW_IMAGE=017690734226.dkr.ecr.us-east-1.amazonaws.com/ecs-deploy:latest
              TASK_DEFINITION=$(aws ecs describe-task-definition --task-definition ecs-demo-2 --region "$AWS_REGION")
              NEW_TASK_DEFINITION=$(echo $TASK_DEFINITION | jq --arg IMAGE "$NEW_IMAGE" '.taskDefinition | .containerDefinitions[0].image = $IMAGE | del(.taskDefinitionArn) | del(.revision) | del(.status) | del(.requiresAttributes) | del(.compatibilities) | del(.registeredAt) | del(.registeredBy)')
              NEW_REVISION=$(aws ecs register-task-definition --region "$AWS_REGION" --cli-input-json "$NEW_TASK_DEFINITION")
              NEW_REVISION_DATA=$(echo $NEW_REVISION | jq '.taskDefinition.revision')
              NEW_SERVICE=$(aws ecs update-service --cluster gitlab-ecs-demo --service ecs-demo-svc-2 --task-definition ecs-demo-2 --force-new-deployment)
               '''
            }
        } 
        stage('done') {
            steps {
                sh '''
                echo ${ecs-demo-2},Revision: ${NEW_REVISION_DATA}
                '''
                
            }     
    }
}
}
