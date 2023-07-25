# Build a Serverless Web Application
## Overview
In this tutorial, you will create a simple serverless web application that enables users to request unicorn rides from the Wild Rydes fleet. The application will present users with an HTML-based user interface for indicating the location where they would like to be picked up and will interact with a RESTful web service on the backend to submit the request and dispatch a nearby unicorn. The application will also provide facilities for users to register with the service and log in before requesting rides.
## Prerequisites
To complete this tutorial, you will need an AWS account, an account with ArcGIS to add mapping to your app, a text editor, and a web browser. If you don't already have an AWS account, you can follow the Setting Up Your AWS Environment getting started guide for a quick overview.
## Application architecture
The application architecture uses AWS Lambda, Amazon API Gateway, Amazon DynamoDB, Amazon Cognito, and AWS Amplify Console. Amplify Console provides continuous deployment and hosting of the static web resources including HTML, CSS, JavaScript, and image files which are loaded in the user's browser. JavaScript executed in the browser sends and receives data from a public backend API built using Lambda and API Gateway. Amazon Cognito provides user management and authentication functions to secure the backend API. Finally, DynamoDB provides a persistence layer where data can be stored by the API's Lambda function.
![Architecture](image.png)
### Static Web Hosting
AWS Amplify hosts static web resources including HTML, CSS, JavaScript, and image files which are loaded in the user's browser.
### User Management
Amazon Cognito provides user management and authentication functions to secure the backend API.
### Serverless Backend
Amazon DynamoDB provides a persistence layer where data can be stored by the API's Lambda function.
### RESTful API
JavaScript executed in the browser sends and receives data from a public backend API built using Lambda and API Gateway.
## Modules
This tutorial is divided into five modules. Each module describes a scenario of what we're going to build and step-by-step directions to help you implement the architecture and verify your work.
- Host a Static Website (15 minutes): Configure AWS Amplify to host the static resources for your web application with continuous deployment built in 
- Manage Users (30 minutes): Create an Amazon Cognito user pool to manage your users' accounts
- Build a Serverless Backend (30 minutes): Build a backend process for handling requests for your web application
- Deploy a RESTful API (15 minutes): Use Amazon API Gateway to expose the Lambda function you built in the previous module as a RESTful API
- Terminate Resources (10 minutes): Terminate all the resources you created throughout this tutorial
### Static Web Hosting with Continuous Deployment
In this module, you will configure AWS Amplify to host the static resources for your web application with continuous deployment built in. The Amplify Console provides a git-based workflow for continuous deployment and hosting of full-stack web apps. In subsequent modules, you will add dynamic functionality to these pages using JavaScript to call remote RESTful APIs built with AWS Lambda and Amazon API Gateway. The architecture for this module is straightforward. All of your static web content including HTML, CSS, JavaScript, images, and other files will be managed by AWS Amplify Console. Your end users will then access your site using the public website URL exposed by AWS Amplify Console. You don't need to run any web servers or use other services to make your site available. For most real applications you'll want to use a custom domain to host your site. If you're interested in using your own domain, follow the instructions for setting up a custom domain on Amplify.
### Implementation
- Select a region
- Create a Git repository
- Populate the Git repository
Change directory into your repository and copy the static files from S3:
```
aws s3 cp s3://wildrydes-us-east-1/WebApplication/1_StaticWebHosting/website ./ --recursive
```
Commit the files to your Git service
```
git add .
git commit -m 'new'
git push
```
- Enable Web Hosting with the AWS Amplify Console
Next you'll use the AWS Amplify Console to deploy the website you've just committed to git. The Amplify Console takes care of the work of setting up a place to store your static web application code and provides a number of helpful capabilities to simplify both the lifecycle of that application as well as enable best practices.\

a. Launch the Amplify Console console page\
b. Click Get Started under Deploy with Amplify Console\
c. Go to New App on the top right and choose Host Web App\
d. Select CodeCommit under Get started with Amplify Hosting\
e. Select the Repository service provider used today and select Continue\
f. If you used GitHub, you'll need to authorize AWS Amplify to your GitHub account\
g. From the dropdown, select the Repository and Branch you just created and select Next\
h. On the Configure build settings page, leave all the defaults, Select Allow AWS Amplify to automatically deploy all files hosted in your project root directory and select Next.\
i. On the "Review" page select Save and deploy\
j. The process takes a couple of minutes for Amplify Console to create the necessary resources and to deploy your code.\
Once completed, click on the site image to launch your Wild Rydes site. If you click on the link for Master you'll see the build and deployment details related to your branch, and screenshots of the app on various devices
- Modify your site
The AWS Amplify Console will rebuild and redeploy the app when it detects changes to the connected repository. Make a change to the main page to test out this process.\
a. From your local machine, open `wildryde-site/index.html` in a text editor of your choice and modify the title line so that it says: <title>Wild Rydes - Rydes of the Future!</title>\
b. Save the file and commit to your git repository again. Amplify Console will begin to build the site again soon after it notices the update to the repository. It will happen pretty quickly! Head back to the Amplify Console page to watch the process.
```
git add index.html 
git commit -m "updated title"
git push
   ```
- Recap
In this module, you've created static website which will be the base for our Wild Rydes business. AWS Amplify Console makes it really easy to deploy static websites following a continuous integration and delivery model. It has capabilities for "building" more complicated Javascript framework based applications and has features such as feature branch deployments, easy custom domain setup, instant deployments, and password protection.
### User Management
In this module you'll create an Amazon Cognito user pool to manage your users' accounts. You'll deploy pages that enable customers to register as a new user, verify their email address, and sign into the site.\
When users visit your website they will first register a new user account. For the purposes of this workshop we'll only require them to provide an email address and password to register. However, you can configure Amazon Cognito to require additional attributes in your own applications.\
After users submit their registration, Amazon Cognito will send a confirmation email with a verification code to the address they provided. To confirm their account, users will return to your site and enter their email address and the verification code they received. You can also confirm user accounts using the Amazon Cognito console with a fake email addresses for testing.\
After users have a confirmed account (either using the email verification process or a manual confirmation through the console), they will be able to sign in. When users sign in, they enter their username (or email) and password. A JavaScript function then communicates with Amazon Cognito, authenticates using the Secure Remote Password protocol (SRP), and receives back a set of JSON Web Tokens (JWT). The JWTs contain claims about the identity of the user and will be used in the next module to authenticate against the RESTful API you build with Amazon API Gateway.\
### Implementation
- Create an Amazon Cognito User Pool and Integrate an App to Your User Pool
Amazon Cognito provides two different mechanisms for authenticating users. You can use Cognito User Pools to add sign-up and sign-in functionality to your application or use Cognito Identity Pools to authenticate users through social identity providers such as Facebook, Twitter, or Amazon, with SAML identity solutions, or by using your own identity system. For this module you'll use a user pool as the backend for the provided registration and sign-in pages.\
a. In the AWS Console, enter Cognito in the search bar and select Cognito from the search results.\
b. Choose Create user pool.\
c. On the Configure sign-in experience page, in the Cognito user pool sign-in options section, select User name. Keep the defaults for the other settings, such as Provider types in the Authentication providers section. Choose Next.\
d. On the Configure security requirements page, keep the Password policy mode as Cognito defaults. You can choose to configure multi-factor authentication (MFA) or choose No MFA and keep other configurations as default. Choose Next.\
e. On the Configure sign-up experience page, keep everything as default. Choose Next.\
f. On the Configure message delivery page, for Email provider, confirm that Send email with Amazon SES - Recommended is selected. In the FROM email address field, select an email address that you have verified with Amazon SES, following these instructions from the Amazon SES Developer Guide.\
g. On the Integrate your app page, provide a name for your user pool such as WildRydes. Under Initial app client, give the app client a name such as WildRydesWebApp and keep other settings as default.\
h. On the Review and create page, choose Create user pool.\
i. Note the Pool ID and the App client ID on the Pool details page of your newly created user pool.
- Update the Website Config
The /js/config.js file contains settings for the user pool ID, app client ID and Region. Update this file with the settings from the user pool and app you created in the previous steps and upload the file back to your bucket\
a. From your local machine, open `wildryde-site/js/config.js` in a text editor of your choice.\
b. Update the cognito section with the correct values for the user pool and app you just created.
You can find the value for userPoolId on the Pool details page of the Amazon Cognito console after you select the user pool that you created.\
You can find the value for userPoolClientId by selecting App clients from the left navigation bar. Use the value from the App client id field for the app you created in the previous section.\
The value for region should be the AWS Region code where you created your user pool. E.g. us-east-1 for the N. Virginia Region, or us-west-2 for the Oregon Region. If you're not sure which code to use, you can look at the Pool ARN value on the Pool details page. The Region code is the part of the ARN immediately after arn:aws:cognito-idp:.\
The updated config.js file should look like this. Note that the actual values for your file will be different:
```
window._config = {
    cognito: {
        userPoolId: 'us-west-2_uXboG5pAb', // e.g. us-east-2_uXboG5pAb
        userPoolClientId: '25ddkmj4v6hfsfvruhpfi7n4hv', // e.g. 25ddkmj4v6hfsfvruhpfi7n4hv
        region: 'us-west-2' // e.g. us-east-2
    },
    api: {
        invokeUrl: '' // e.g. https://rc7nyt4tql.execute-api.us-west-2.amazonaws.com/prod',
    }
};
```
d. Save the modified file and push it to your Git repository to have it automatically deploy to Amplify Console.\
```
git add .
git commit -m "new_config"
git push
```
- Validate your implementation
