Run details
Workflow file for this run
.github/workflows/cicd-workflow.yml at d70e03b
name: CICD

on:
  push:
    branches: 
      - mern-ec2-docker

jobs:
  build:
    runs-on: [ubuntu-latest]
    steps:
      - name: Checkout source
        uses: actions/checkout@v4
      - name: Login to docker hub
        run: docker login -u ${{ secrets.DOCKER_USERNAME }} -p ${{ secrets.DOCKER_PASSWORD }} 
      - name: Build docker image
        run: docker build -t integrationninjas/nodejs-app .
      - name: Publish image to docker hub
        run: docker push integrationninjas/nodejs-app:latest

  deploy:
    needs: build
    runs-on: [self-hosted]
    steps:
      - name: Pull image from docker hub
        run: docker pull integrationninjas/nodejs-app:latest
      - name: Delete old container
        run: docker rm -f nodejs-app-container
      - name: Run docker container
        run: docker run -d -p 4000:4000 --name nodejs-app-container -e MONGO_PASSWORD='${{ secrets.MONGO_PASSWORD }}' integrationninjas/nodejs-app
