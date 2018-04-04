# Lab 7 - Container Healthchecks - Detailed Steps

## 7.1	Update the task definition

Up to this point the ECS service scheduler relied on the Elastic Load Balancer (ELB) to report container health status and to restart any unhealthy containers. In this step, you will create a new revision of the Dogs task definition to include a Docker health checks. This health check will 

1. Sign-in to the AWS management console and open the Amazon ECS console at https://console.aws.amazon.com/ecs/.

2. Select **Task Definitions** from the left-hand menu.

3. Select **dogs** from the list of task definitions.

5. Select the most revcent revision of the **dogs** task definition.

6. Click **Create new revision**

7. Under **Container Definitions** click on **dogs**.

8. Under **HEALTHCHECK**:
    
    1. In **Command** enter: `[ "CMD-SHELL", "curl -f http://localhost/dogs/1.jpg || exit 1" ]`
    
    2. In **Interval** enter: *30*
    
    3. In **Timeout** enter: *5*
  
    4. In **Start period** enter: *0*
    
    5. In **Retries** enter: *3*
    
    When the container starts it runs a curl command to test for the presence of localhost/dogs/1.jpg. Any response code other than a 200 is considered a failure. The test will timeout after 5 seconds, at which point it will wait for 30 seconds and try again. There will be a total of 3 attempts to execute this health check, for a commulative time of 90 seconds (3 * 20). At which point, the task will be marked as unhealthy, it will be removed from the service and replaced.

9. Click **Update**

10. Click **Create**

## 7.2	Update the service

The next step is to update the dogs service definition to use the latest revision of the task definition which includes the health check. 

1. In the navigation pane choose **Clusters**.

2. Click the cluster **catsndogsECScluster**.

3. Click the Services tab then click the **dogs** service.

4. Click **Update**.

5.	In Configure service:

    1. In **Task definition**, choose the task definition you created in the earlier step.
    
    2. In **Cluster**, choose the cluster **catsndogsECScluster**.

6.	Click **Next step**.

7.	In **Load Balancing**, choose **Next step**.

8.	In **Service Auto Scaling (optional)**, click **Next step**.

9.	Review the settings, and click **Update service**.

10.	Click **View Service**.

## 7.3	Inject a bug

In this step, we will intentionally inject a fault in to the configuration of the Dogs application and observe container health checks helps us ensure that bugs have minimal impact on the availability of our applictions.

1. Sign in to the AWS management console, click on **Services**.

2. In the **Management Tools** section, click **CloudFormation**.

3. Click on the stack **catsndog-ide**.

4. Expand **Outputs**, locate the **Cloud9IDE** output. Click on the associated link to launch the Cloud9 IDE.

5. At the command prompt run `cd ~/environment/dogs` to switch to the local clone of the Dogs application repository.

6. Run the command `nano Dockerfile` to edit the Dockerfile.

7. Find the line which starts with `RUN aws --region` and add a comment out that line by adding a `#` before the word `RUN`. The line should look somehing like this: 

    `# RUN aws --region us-west-2 s3 cp s3://catsndogs-assets/dogs-images /www/ --recursive`
    
During the creation of the container image, this line is respinsible for fetching the dog meme images from a source S3 bucket and writing them in to the local file system of the container image so that they can be served by the Dogs application. By commenting out this line, there will be no images written to the container image and as such, our Docker health check should fail.

8. Within the nano editor press `ctrl + x` to exit the editor. When prompted type `Y` to confirm that the changes should be saved.

9. Commit the changes that have just been made t and push them to the remote repository by running the following commands:

    1.	`git add Dockerfile`

    2.	`git command -m ‘Injecting a failure’`
    
    3.	`git push`
    
10.	Open the AWS management console, and open the **AWS CodePipeline** console at https://console.aws.amazon.com/codepipeline/.

11.	Verify that the pipeline started:

    1.	From the **All Pipelines** table, click the **CatsnDogsPipeline**, to monitor the progress of your pipeline.

    2.	The status of each stage should change from No executions yet to **In progress**.
    


## 7.4  Observe the deployment
