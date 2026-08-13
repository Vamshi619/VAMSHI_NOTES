[1. Moving a container image into Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html)

- step a. Once we create our project with docker , we need to create a docker image
- [step b. Then we need to create a Repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-create-repository)
- [step c: is to Authenticate to our default Registry](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-authenticate-registry)
- [Step d: And then Push an image into Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-push-image)
- Optional : [Pulling an image from Amazon ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-pull-image)
- Optional : [Deleting an image](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-delete-image)
- Optional : [Deleting a Repository](https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html#cli-delete-repository)

-----
1. To create a Repository
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

