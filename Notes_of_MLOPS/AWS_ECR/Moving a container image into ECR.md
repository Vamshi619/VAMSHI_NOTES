[1. Moving a container image into Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html)

- step a. Once we create our project with docker , we need to create a docker image
- [step b. Then we need to create a Repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-create-repository)
- [step c: is to Authenticate to our default Registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-authenticate-registry)
- [Step d: And then Push an image into Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-push-image)
- Optional : [Pulling an image from Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-pull-image)
- Optional : [Deleting an image](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-delete-image)
- Optional : [Deleting a Repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-delete-repository)

-----
**1. To create a Repository**
   - We usually need to define the Repository name and the region of it
   - so we can use the below command
```
aws ecr create-repository \
    --repository-name <repository name> \
    --region <region>
```
- once we run this command a Repository gets created and we get
      <img width="1035" height="328" alt="image" src="https://github.com/user-attachments/assets/4855383b-259b-430a-a6a0-1fe974a967d6" />
      
- By this in the Amazon ECR our new Repository gets created

  <img width="1743" height="490" alt="image" src="https://github.com/user-attachments/assets/7857c6e9-bd37-4824-942a-e6e283bd253a" />

-  So once you click the repository you will be able to see the images(container images) in it

      <img width="1743" height="490" alt="image" src="https://github.com/user-attachments/assets/a9087e7f-b85f-4bdc-8751-d92e242c6cb3" />

**2. Next if you want to know the User ID , Account and ARN for Authenticating to default registry**
```
   aws sts get-caller-identity
```

   - <img width="1188" height="142" alt="image" src="https://github.com/user-attachments/assets/2cf3bd7f-ce78-45ab-98f0-e23dbb688c7b" />

**3. Now Authenticating to Default Regstry**

   - we need to use this command

```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```

   - For example

```
aws ecr get-login-password --region ap-south-1 | \
docker login \
--username AWS \
--password-stdin \
121470661513.dkr.ecr.ap-south-1.amazonaws.com
```
   - once we do this we usually get logged in to the docker
      <img width="1201" height="226" alt="image" src="https://github.com/user-attachments/assets/fb6698f3-0699-43e3-9360-7f4cde3739f3" />

**4. Step 4: Pushing image into Amazon ECR**
   - Here when ever we create a project, we usually create a docker and ECR is a registry that helps to store the image into it so that when ever we want to test it we can pull the image into our local and run it in our local , if not if we want to make the project serve multiple people then we want to pull the image into ECS or EKS or EC2 like that then we can directly pull the image from the ECR.
   - Before pushing it into ECR better to check the current docker images
     ```
     docker images
     ```
   - Then we can create an ECR style tag for the local image (as we need to first tag the image to push it into repository)

     ```
     docker tag hello-world:latest aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository
     ```
     ```
      docker tag customer-churn-api:v1 \
      121470661513.dkr.ecr.ap-south-1.amazonaws.com/mlops_practice:v1

   - <img width="1505" height="309" alt="image" src="https://github.com/user-attachments/assets/ee8d7fb0-9803-4d8a-97d4-75e466a589b4" />

   - Then we can push the Image into ECR

     ```
     docker push aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository:latest
     ```

     ```
     docker push 121470661513.dkr.ecr.ap-south-1.amazonaws.com/mlops_practice:mlops_practice
     ```

- Once we push the image into ECR we usually get image digest whcih is a unique key value that works like a finger print for the docker image, created with the help of Hash Algorithm
- we can check even in the list of images showing commands as well in ECR.
    

    <img width="1499" height="382" alt="image" src="https://github.com/user-attachments/assets/1a1bf271-d459-47da-b9c6-9ea5f3107d52" />

   **5. Pull the Docker image and Run**

    ```
    docker pull aws_account_id.dkr.ecr.region.amazonaws.com/hello-repository:latest 
    ```
    ```
    docker pull 121470661513.dkr.ecr.ap-south-1.amazonaws.com/mlops_practice:v1
    ```

    **6. Running the Docker image after pulling**
     - <img width="1501" height="92" alt="image" src="https://github.com/user-attachments/assets/293e79ae-c2af-4cf6-92af-8a195af68d6e" />
     - Usually first is the port number where our swagger or local ui api is running and next is like 8000 is the docker port
        ```
        docker run -d   -p 8002:8000   customer-churn-api:v1
        ```
    **7. Deleting Image and Deleting a Repository**
  
        
        aws ecr batch-delete-image \
      --repository-name hello-repository \
      --image-ids imageTag=latest \
      --region region


-       aws ecr delete-repository \
      --repository-name hello-repository \
      --force \
      --region region

        








