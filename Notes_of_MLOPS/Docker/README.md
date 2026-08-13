#**Docker Commands**

**To check logs of container:** 

```
docker logs <container-name>
```

**To follow logs of the container live:**

```
docker logs -f <container-name>
```

**To Stop the Container:**

```
docker stop <container-name>
```

**To the Start the Docker Container again:**
```
docker start <container-name>
```

**Remove the COntainer:**
```
docker rm customer-churn-container
```

**Remove the Image:**
```
docker rmi customer-churn-api:v1
```

**TO build the docker container:**
```
docker build -t customer-churn-api:v1 .
```

**To check all the docker images:**
```
docker images
```

**To check no of containers:**

```
docker ps
```





--------


