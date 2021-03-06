# Worker Safety with AWS DeepLens and Amazon Rekognition

## Learning Objectives of This lab
In this lab you will be achieving the following:
- Create and deploy object detection project to DeepLens.
- Modify the DeepLens object detection inference lambda function to detect persons and upload frame to S3.
- Create lambda function to identify persons who are not wearing safety hats.
- Analyze results using IoT and CloudWatch and Web Dashboard.
## Table of Contents
1. [Architecture](#Architecture)
2. [Modules](#Modules)
3. [LAB 1 - Register your DeepLens Device](#registerdl)
4. [LAB 2 - Create an *object-detection* project](#createmodel)
    - [Deploy your project](#deployproject)
    - [View your project output](#projectoutput)

5. [LAB 3 - Create Worker Safety Project](#workersafetyproject):
    - [Step 1: Setup IAM Role for Cloud Lambda](#cloudiamrole)
     - [Step 2: Setup IAM Role for DeepLens Lambda](#dliamrole)
     - [Step 3: Create S3 bucket](#s3create)
    - [Step 4: Create Cloud Lambda](#cloudlambda)
    - [Step 5: Create DeepLens Inference Lambda Function](#inferencelambda)
    - [Step 6: Create DeepLens Project](#createdlproject)
    - [Step 7: Deploy DeepLens Project](#deploydlproject)
     - [Step 8: View Output in IoT](#iotoutput)
     - [Step 9: View Output in CloudWatch](#cloudwatchoutput)
    - [Step 10: View Output in Web Dashboard](#dashboardoutput)
6. [Clean Up](#cleanup)

## Architecture

![](arch.png)

## Modules

**Note:** *To keep this page open while following the instructions, hold down the control button (CRTL) and click on the link for Windows machine or command + click on Mac OS machine*

### Lab 1 - Register your DeepLens Device <a id="registerdl"></a>
If you recycle a device from another user, make sure that the previous user has deregistered the device before registering it again.

#### Configure your AWS account for AWS DeepLens

1. Sign in to the AWS Management Console for AWS DeepLens at <a href="https://console.aws.amazon.com/deeplens/home?region=us-east-1#firstrun" target="_blank">AWS Managment Console</a>.


2. Choose Register device. If you don't see a Register device button, choose Devices on the main navigation pane.

3. In the Name your device section on the Configure your AWS account page, type a name (e.g., lab1-deepLens) for your AWS DeepLens device in the Device name text field .
4. The device name can have up to 100 characters. Valid characters are a-z, A-Z, 0-9, and - (hyphen) only. 
5. In the Permissions section, choose Create roles for the AWS DeepLens console to create the required IAM roles with relevant permissions on your behalf.
- After the roles are successfully created, you'll be informed that you have the necessary permissions for setting the AWS DeepLens device. If the roles already exist in your account, the same message will be displayed.
6. In the Certificate section, choose Download certificate to save the device certificate.
7. The downloaded device certificate is a .zip file. Don’t unzip it.

**Important**

Certificates aren't reusable. You must generate a new certificate every time you register your device.

8. After the certificate is downloaded, choose Next to proceed to joining your computer to your device's (AMDC-NNNN) Wi-Fi network in order to start the device setup application hosted on the device. 
9. Plug in your AWS DeepLens device to an AC power outlet. Press the power button on the front of the device to turn the device on. 
10. Wait until the device has entered into setup mode when the Wi-Fi indicator (middle LED) on the front of the device starts to flash. 
    - *Note*: If Wi-Fi indicator (middle LED) does not flash, the device in no longer in the setup mode. To turn on the device's setup mode again, press CAREFULLY a paper clip into the reset pinhole on the back of the device. After you hear a click, wait about 20 seconds for the Wi-Fi indicator to blink. 
11. Open the network management tool on your computer. Choose your device's SSID from the list of available Wi-Fi networks and type the password for the device's network. The SSID and password are printed on the bottom of your device. The device's Wi-Fi network's SSID has the AMDC-NNNN format
12. After successfully connecting your computer to the device's Wi-Fi network, you're now ready to launch the device setup application to configure your device. 


### LAB 2 - Create an *object-detection* project using DeepLens<a id="createmodel"></a>

#### Create Your Project

1. Using your browser, open a **new tab** for AWS DeepLens console at https://console.aws.amazon.com/deeplens/.
2. Choose Projects, then choose Create new project.
3. On the Choose project type screen
- Choose Use a project template, then choose Object detection.

![](assets/projecttemplate.png)

- Scroll to the bottom of the screen, then choose Next.
4. On the Specify project details screen
   - In the Project information section:
      - Either accept the default name for the project, or type a name you prefer.
      - Either accept the default description for the project, or type a description you prefer.

  - Choose Create.

This returns you to the Projects screen where the project you just created is listed with your other projects.

#### Deploy your project <a id="deployproject"></a>

Next you will deploy the Object Detection project you just created.

1. From Deeplens console, On the Projects screen, choose the radio button to the left of your project name, then choose Deploy to device.

![](assets/projecthome.png)

2. On the Target device screen, from the list of AWS DeepLens devices, choose the radio button to the left of the device that you want to deploy this project to. An AWS DeepLens device can have only one project deployed to it at a time.

![](assets/targetdevice.png)

3. Choose Review.

   This will take you to the Review and deploy screen.

   If a project is already deployed to the device, you will see an error message
   "There is an existing project on this device. Do you want to replace it?
   If you Deploy, AWS DeepLens will remove the current project before deploying the new project."

   If you receive an error message stating "Cloud not find a project" please omit and proceed.

4. On the Review and deploy screen, review your project and choose Deploy to deploy the project.

   This will take you to to device screen, which shows the progress of your project deployment.

#### View your project log messages in IoT:

You can also view the log messages that your project's Lambda function running on DeepLens device sends to IoT topic.

1. Once the deployment has been successful (Green message at the top), click on *Devices* the left menu.
2. Click on the name of your DeepLens device and on the next screen click on Copy button on the IoT topic under Project ouput.

![](assets/dliottopic.png)

3. On a new browser tap open the AWS IoT Console at https://console.aws.amazon.com/iot/home?region=us-east-1#/dashboard
4. Click on Test in the left navigation.
5. Paste the IoT topic in the textbox under Subscription topic and click Subscribe to topic
6. You should now see log messages published from DeepLens device to IoT.

![](assets/dlmessages.png)

### Completion:
You have created and deployed object detection project to your Deeplens device.

### LAB 3 - Create and Deploy Worker Safety Project <a id="workersafetyproject"></a>

#### Step 1: Setup IAM Role for Cloud Lambda <a id="cloudiamrole"></a>

1. Go to IAM in AWS Console at https://console.aws.amazon.com/iam
2. Click on Roles
3. Click create role
4. Under AWS service, select Lambda and click Next: Permissions
5. Under Attach permission policies
    1. search S3 and select *AmazonS3FullAccess*
    2. search Rekognition and select checkbox next to *AmazonRekognitionReadOnlyAccess*
    3. search cloudwatch and select checkbox next to *CloudWatchLogsFullAccess* and *CloudWatchFullAccess*
    4. search iot and select *AWSIotDataAccess*
    5. search lambda and select checkbox next to *AWSLambdaFullAccess*
6. Click Next: Tags and Next: Review
7. Name the role “*RecognizeObjectLambdaRole*”
8. Click Create role


#### Step 2: Setup IAM Role for DeepLens Lambda <a id="dliamrole"></a>

1. Click create role
2. Under AWS service, select Lambda and click Next: Permissions
3. Under Attach permission policies
    1. search S3 and select AmazonS3FullAccess
    2. search lambda and select checkbox next to AWSLambdaFullAccess
4. Click Next: Tags and Next: Review
5. Name is “DeepLensInferenceLambdaRole”
6. Click Create role


#### Step 3: Create S3 bucket <a id="s3create"></a>

1. Go to Amazon S3 in AWS Console at https://s3.console.aws.amazon.com/s3/
2. Click on Create bucket.
3. Under Name and region:

* Bucket name: Enter a bucket name- your name-worker (example: lab1-worker)
* Choose US East (N. Virginia)
* Click Next

1. Leave default values for Configure Options screen and click Next
2.  Under Set permissions, uncheck all four checkboxes. NOTE: This step would allow us to make objects in your S3 bucket public. We are doing this to reduce few steps in the module, but you should not do that for production workloads. Instead it is recommended to use S3 Pre-Signed URLs to give time limited access to objects in S3.
3. Click Next, and click Create bucket.

- Add the proper policy in order to allow DeepLens inference Lambda to upload images to S3:
4. Go to IAM in AWS Console at https://console.aws.amazon.com/iam
5. Click on Roles
6. Search for "*AWSDeepLensGreengrassGroupRole*" and click on the role
7. Click on Attach Policies
8. Search for "*AmazonS3FullAccess*", click on the checkbox and click on Attach Policy

#### Step 4: Create Cloud Lambda <a id="cloudlambda"></a>

1. In a new browser tab, go to Lambda in AWS Console at https://console.aws.amazon.com/lambda/
2. Click on Create function.
3. Under Create function, Author from scratch should be selected as default.
4. Under Author from scratch:

* Name: worker-safety-cloud
* Runtime: Python 3.7
* Under permissions, expand Choose or Create and execution role
* Under Execution role, choose "Use an existing role"
* Under existing role choose: RecognizeObjectLambdaRole
* Click Create function

5. Scroll down to Environment variables, add a variable:

* Key: iot_topic
* Value: worker-safety-demo-cloud

6. On a new tab click on [lambda.zip](./code/lambda.zip) and click on *Download* button the next page.
7. Once the download is completed return to the Lambda tab and Under Function code:

* Code entry type: Upload a zip file
* Under Function package, click Upload and select the zip file you downloaded in earlier step.
* Click Save.

8. Scroll at the top of the screeen and under Add triggers, select S3.
9. Under Configure triggers:
* Select S3
* Bucket: Select the S3 bucket you just created in earlier step (example: *lab1*-worker).
* Event type: All Object create events
* Leave defaults for Prefix and Suffix and make sure Enable trigger checkbox is checked.
* Click Add.
* Click Save on the top right to save changed to Lambda function.

#### Step 5: Create DeepLens Inference Lambda Function <a id="inferencelambda"></a>

1. Go to Lambda in AWS Console at https://console.aws.amazon.com/lambda/.
2. Click on Create function.
3. Under Create function, select Blueprints.
4. Under Blueprints, type greengrass and hit enter to filter blueprint templates.
5. Select greengrass-hello-world and click Configure.
6. Under Basic information:

* Name: *lab#*-worker-safety-deeplens (example: lab1-worker-safety-deeplens)
* Under execution role: Choose use an existing role
* Under existing role: Choose DeepLensInferenceLambdaRole
* Click Create function.

7. On a new browser tab click on [deeplens-lambda.py](./code/deeplens-lambda.py):
    - Click on the "*Raw*" button.
    - Select the entire code and copy.
    - Go back to the Lambda Management console tab and under function code replace the existing code with the code copied in the previous step.
8. Go to line 34 of the new code and replace "REPLACE-WITH-NAME-OF-YOUR-S3-BUCKET" with "*lab#*-worker" (example: lab1-worker)

9. Click Save.
10. Click on Actions, and then "Publish new version".
11. For Version description enter: Detect person and push frame to S3 bucket. and click Publish.

#### Step 6: Create DeepLens Project <a id="createdlproject"></a>

1. On new tab, open the AWS DeepLens console at https://console.aws.amazon.com/deeplens/.
2. Choose Projects, then choose Create new project.
3. On the Choose project type screen

* Choose Create a new blank project, and click Next.

4. On the Specify project details screen

    * Under Project information section:
        * Project name: your-user-name-worker-safety (example: lab1-worker-safety)
    * Under Project content:
        * Click on Add model, click on radio button for deeplens-object-detection and click Add model.
        * Click on Add function, click on radio button for your lambda function (example: lab1-worker-safety-deeplens) lambda function and click Add function.
* Click Create. This returns you to the Projects screen.

#### Step 7: Deploy DeepLens Project <a id="deploydlproject"></a>

1. From DeepLens console, On the Projects screen, choose the radio button to the left of your project name, then choose Deploy to device.
2. On the Target device screen, from the list of AWS DeepLens devices, choose the radio button to the left of the device where you want to deploy this project.
3. Choose Review. This will take you to the Review and deploy screen.
    If a project is already deployed to the device, you will see a warning message "There is an existing project on this device. Do you want to replace it? If you Deploy, AWS DeepLens will remove the current project before deploying the new project."
4. On the Review and deploy screen, review your project and click Deploy to deploy the project. This will take you to to device screen, which shows the progress of your project deployment.

#### Step 8: View Output in IoT <a id="iotoutput"></a>

1. On new tab, open the AWS IoT Console at https://console.aws.amazon.com/iot/home
2. Under Subscription topic enter topic name *worker-safety-demo-cloud* and click Subscribe to topic.
3. You should now see JSON message with a list of people detected and whether they are wearing safety hats or not.

#### Step 9: View Output in CloudWatch <a id="cloudwatchoutput"></a>

1. On new tab, open the AWS CloudWatch Console at https://console.aws.amazon.com/cloudwatch
2. Go to Dashboard on the left menu and click create dashboard called “*lab#*-worker-safety-dashboard”
3. In the popup window select widget type, *Line* and click configure
4. Under Custom Namespaces, select “string”, “Metrics with no dimensions”, and then select PersonsWithSafetyHat and PersonsWithoutSafetyHat. And click on create widget.
5. In the dropdown menu on top right, check the “Auto-refresh” option and set to 10 seconds.
6. Review the dashboard graph

## Lab 4 - View Output in Web Dashboard <a id="dashboardoutput"></a>

1. On new tab, open the AWS S3 Console at https://console.aws.amazon.com/s3
2. Open the bucket (folder) called *lab#*-worker-web (example: *lab1*-worker-web)
3. Click on index.html and click on the link at the bottom under "Open URL" to open the web page in browser.
4. In the address URL append ?iottopic=worker-safety-demo-cloud. This is the same value you added to Lambda environment variable and hit Enter.
5. You should now see images coming from DeepLens with a green or red box around the person and detecting whether the peron is wearing a safety helmet


## Clean Up <a id="cleanup"></a>
Delete Lambda functions, S3 bucket and IAM roles.
