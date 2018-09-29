# aws-lambda-python-s3
A sample AWS Lambda Python function to listen to AWS S3 event through AWS SNS and access the object from AWS using boto3 and downsize it using Pillow (Python Imaging Library). It will be used as the sample application to demonstrate the

# AWS Lambda Requirements:
- 256 MB of Memory. (Can try with lower one)
- Timeout 2 min.
- Execution role
-- Access to S3 objects Policy

# Libraries
- boto3: https://github.com/boto/boto3
- Pillow: https://github.com/python-pillow/Pillow
- localstack: https://github.com/localstack/localstack
- aws-sam-cli: https://github.com/awslabs/aws-sam-cli

# Compile dependency libraries for Python function
- Install Docker on your machine if you do not have one: https://www.docker.com/get-docker
- As mentioned in the ref.[1], AWS Lambda function is executed in Ubuntu OS. Therefore, you have to ensure that the libraries that you downloaded for the Python function is compatible with the Lambda function OS env.
- To do that, we will spin up an Ubuntu container using any Ubuntu image you like:
  - docker run -v <python-function-full-path-directory>:/data -it -d --name lambda-build-env ubuntu
- You will see an auto-generated id when the container is created.
- Then, you can use the auto-generated id or the docker name to access the container
  - docker exec -it lambda-build-env /bin/bash
- Once you are in the container, execute the following functions to install python-pip:
  - apt-get update
  - apt-get install python-pip
- After that, perform the following to cd to the working directory and install the required dependencies to the dist folder depending on the pip-req.txt. And also copy the generate-thumbnails.py to the dist folder.
  - cd /data/
  - pip install -r pip-req.txt -t dist/
  - cp generate-thumbnails.py dist/
- After that, you should see some folders and the generate-thumbnails.py in the dist folder. Exit the container by:
  - exit
- Next will be to install sam-local and localstack on your local environment.

# Prepare for local deployment using sam-local and localstack
- Install Docker on your machine: https://www.docker.com/get-docker
- Install aws-sam-local following the guide on the github readme.
- Clone the localstack repo to your machine. Then, navigate to the local branch folder to spin up the Localstack using docker-compose: TMPDIR=/private$TMPDIR docker-compose up
  - You should seet "localstack_1 | Ready " when the localstack container is spinned up.
  - Then, you should check the network created for localstack container. The default is: localstack_default.
    - Or you run the command: docker inspect <CONTAINER_ID> -f "{{json .NetworkSettings.Networks }}".

# AWS Lambda execution using sam-local and localstack(S3)
- To demonstrate the actual use case of using AWS lambda function to access other AWS services, we will use S3 provided by localstack.
- Now, you will need to create bucket and upload a photo to the bucket:
  - Create bucket:
    - aws --endpoint-url=http://localhost:4572 s3 mb s3://photo
  - Copy photo to bucket:
    - aws --endpoint-url=http://localhost:4572 s3 cp ./sample/shiba-inu.jpg s3://photo/raw/
- Then, navigate to the folder where the SAM template resides and execute:
  - sam local invoke GenerateThumbnailFunction --log-file ./output.log -e event.json --docker-network <LOCALSTACK NETWORK>
    - Since the aws lambda function is executed in a docker container, it cant access the localstack deployed on the host machine. That is the reason you will need to deploy the container containing the lambda function to the same network as the localstack.
    - You can check on the output.log for debuging purpose.
    - There is also another way to pass trigger event to the lambda. You can read more from the sam-local github page.
- You can sync the photo from the S3 bucket to your local to verify that the thumbnail has been generated.
  - aws --endpoint-url=http://localhost:4572 s3 sync s3://photo/ ./sample/

# Credits
- Cher Ern, my ex-coworker who has written the original code. (the current one is the slim version for demo purpose.)

# Reference
- For setting up local AWS cloud stack: https://github.com/localstack/localstack
- CLI tool for local development and testing of AWS lambda: https://github.com/awslabs/aws-sam-local
- The article which suggested that libraries used by python application should be compiled in Ubuntu environment: https://medium.freecodecamp.org/escaping-lambda-function-hell-using-docker-40b187ec1e48
- Localstack command reference: https://lobster1234.github.io/2017/04/05/working-with-localstack-command-line/
- Localstack S3 command reference: http://bluesock.org/~willkg/blog/dev/using_localstack_for_s3.html
