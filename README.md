# CLO835-Assignment1-Application

#Workflow Content

name: Deploy to ECR

on: 
  push:
    branches: [ main ]

jobs:
  
  build:
    
    name: Build Image
    runs-on: ubuntu-latest

   
    steps:

    - name: Check out code
      uses: actions/checkout@v4

    - name: Login to Amazon ECR
      id: login-ecr
      uses: aws-actions/amazon-ecr-login@v1
      env:
        AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
        AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        AWS_SESSION_TOKEN: ${{ secrets.AWS_SESSION_TOKEN }}
        AWS_REGION: us-east-1

    - name: Build, test, tag, and push image to Amazon ECR
      env:
        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
        ECR_REPOSITORY: jg-asg1-ecr
        IMAGE_TAG: 1.0T
        # IMAGE_TAG: ${{ github.run_number }}
      run: |
        ls -ltra

        # Docker build command
        docker build -t jg_app -f Dockerfile .
        docker build -t jg_db -f Dockerfile_mysql .

        export ECR=$ECR_REGISTRY/$ECR_REPOSITORY

        # Deploy database
        docker run -d --name jg_db -e MYSQL_ROOT_PASSWORD=wordpass  jg_db

        # # Setup env variables
        export DBHOST=127.0.0.1
        export DBPORT=3306
        export DBNAME=Employees
        export DBUSER=root
        export APP_COLOR=blue


        # Deploy app when database is ready
        docker run -d --name jg_app -p 8080:8080  -e DBHOST=$DBHOST -e DBPORT=$DBPORT -e  DBUSER=$DBUSER -e DBPWD=$DBPWD  -e APP_COLOR="blue" jg_app
        
        docker ps
        echo "Pause for 10 seconds to ensure container is deployed"
        sleep 10

        # Test if app is functioning
        netstat -ln | grep 8080
        sudo netstat -tulpn
        sleep 5
        # curl 127.0.0.1:8080 -vvv
        # curl localhost:8080 -vvv

        # Push images when successful
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:app$IMAGE_TAG
        docker push $ECR_REGISTRY/$ECR_REPOSITORY:db$IMAGE_TAG
