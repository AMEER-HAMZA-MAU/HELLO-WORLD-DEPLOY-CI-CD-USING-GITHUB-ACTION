Hello World cicd-app


[![HELLO-WORLD-BY-GITHUB-ACTION.png](https://i.postimg.cc/Sx2pDx1r/HELLO-WORLD-BY-GITHUB-ACTION.png)](https://postimg.cc/NKcV0Bc9)

[![github-action-workflow.png](https://i.postimg.cc/2SVCMwyn/github-action-workflow.png)](https://postimg.cc/hXRF7V9G)

[![github-action-workflow-S.png](https://i.postimg.cc/FFWq8QLn/github-action-workflow-S.png)](https://postimg.cc/kDSfRzwx)


Create an AWS EC2 Instance
AWS EC2 is a cloud computing service that allows us to launch and manage virtual machines known as instances in the cloud, hence providing flexible and scalable computing resources. It allows us to pay for only resources used, making it cost-effective for a wide range of applications and workloads.

Follow these simple steps to create an EC2 instance:

Sign in to AWS Management Console and go to the EC2 dashboard.
aws-console
Click on the Launch instance button.

Set up the instances and configure it to meet the needs of our application. Fill in the following information:
Name: node-cicd-app
Application and OS Images (Amazon Machine Image): ubuntu
ec2-name-os
Instance type: t2.micro (Free tier)
t2-aws-micro
Create Key Pair: cicd-key
aws-pem-key aws-pem-key2
Click on the Create key pair button. AWS will generate and download a key pair .pem file into our computer. This key pair includes a public key for the EC2 instance and a private key to be kept locally.

This key pair allows a secure SSH connection to our EC2 instance. SSH (Secure Shell) is a communication protocol for remote server access and management. The private key ensures encrypted authentication, enhancing security compared to password-based access, and preventing unauthorized entry.

Remember to keep the private key safe and not share it with others, as it grants access to the EC2 instance.

Next, click on the Launch instance button to create our EC2 virtual machine.
ec2-instance-launch
Next, we need to configure our security groups. Security groups are essential to control inbound traffic to our EC2 instance. Security groups act as virtual firewalls, allowing us to specify which ports and protocols are accessible to our instances from different sources (e.g., specific IP addresses, ranges, or the internet).
To set up a security group for our instance:

Select the newly created instance for which you want to configure the security group.
In the tabs below, click Security. Then click on the Security groups link associated with the instance.
security-group
In the Inbound rules tab of the security group, click on Edit inbound rules
security-group2
Add new security rules by specifying the protocol, port range, and source to allow inbound traffic on the necessary ports. Click on the Save rules button to save the security group.
security-group3
Above, we set up a Custom TCP security group rule for port 3000, allowing access from anywhere. This restricts inbound traffic to the necessary connections, enhancing application security against unauthorized access and potential threats.

Next, we’ll create a GitHub action workflow outlining the CI/CD steps to be executed when changes are pushed to our GitHub repository.

Create a Node.Js Github Actions Workflow
A GitHub Actions workflow automatically triggers necessary deployment steps on new code pushes or changes. It executes tasks defined in the workflow configuration. GitHub logs the workflow progress for monitoring.

In case of an error or failure, a red check mark appears in the logs, indicating an issue. Developers can review the log, fix the problem, and push changes to trigger a new workflow. A green check mark confirms a smooth workflow with successful tests and deployment. This visual feedback system ensures our codebase’s health and verifies the application’s functionality.

GitHub offers pre-built workflow actions for common problems. For our article, we’ll use the “Publish Node.js package” template, designed for Node.js projects. With this action, we can easily install dependencies, run tests, and deploy our Node.js application with minimal configuration.

To set up a workflow for our Node.js application, follow these steps:

Access the GitHub repository where the Node.js application resides.
In the repository, navigate to the Actions tab.
Search for node.js action workflow.
Click on the Configure button.
github-action
This will generate a .github/workflows directory to store all our application’s workflows. It will also create a .yml file within this directory where we can define our specific workflow configurations.

Replace the generated .yml file content with the commands below:

name: Node.js CI/CD

on:
  push:
    branches: [ "main" ]

jobs:
  build:
    runs-on: self-hosted
    strategy:
      matrix:
        node-version: [18.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js ${{ matrix.node-version }}
      uses: actions/setup-node@v3
      with:
        node-version: ${{ matrix.node-version }}
        cache: 'npm'
    - run: npm ci
    - run: npm test
    - run: pm2 restart backendserver
In the YAML file above:

Our workflow is named “Node.js CI/CD”.
It triggers when there is a push event to the main branch.
The build job is defined to run on a self-hosted runner. A self-hosted runner is a computing environment that allows us to run GitHub Actions workflows on our own infrastructure instead of using GitHub’s shared runners. With a self-hosted runner, we have more control over the environment in which our workflows are executed.
The steps section lists individual tasks to be executed in sequence.
actions/checkout@v3 fetches the source code of our repository into the runner environment.
actions/setup-node@v3 simplifies Node.js setup on the runner environment for our workflow.
npm ci installs project dependencies. This command performs a clean installation, ensuring consistency for our CI server.
npm test runs tests for our application.
pm2 restart backendserver restarts our server using the PM2 library, which acts as a production process manager. PM2 ensures our Express application runs as a background service and automatically restarts in case of failures or crashes.
The above workflow performs both Continuous Integration (CI) tasks (clean installation, caching, building, testing) and Continuous Deployment (CD) tasks (restarting the server using PM2).

Now, Click the Commit changes button. This will save the modified YAML file to our repository.

Next, return back to the Actions tab on GitHub. Here, we can monitor the workflow in real time and observe logs as each step is been executed on the server.

However, it’s important to note that the above workflow job will fail because we haven’t connected our AWS EC2 instance to the Git repository.

To use our GitHub Actions workflow with an AWS EC2 instance, we must establish a connection between the GitHub repository and the AWS EC2 instance. This connection can be achieved by setting up Git Action Runner on the AWS EC2 instance. This Runner acts as a link between the repository and the instance, enabling direct workflow execution.

To resolve the failed workflow, we’ll connect to our EC2 instance via SSH, locally download and configure the Git Action Runner, and then set up our Node.js application environment on the EC2 instance.

Connect to AWS EC2 Instance via SSH
To install and configure Git Action Runner on our AWS EC2 instance, we start by establishing a local connection to the EC2 instance using the .pem key we previously downloaded. The .pem key serves as the authentication mechanism for securely accessing the EC2 instance through SSH.

Here are the steps to connect to an EC2 instance via SSH:

Open a terminal or command prompt on your local machine.
Ensure you are in the correct directory where the .pem file is located.
Next, head to AWS Management Console and open the newly created instance. Click on the Connect button.
aws-ssh-connect
Next copy and run the chmod command in A in the terminal to confirm if the .pem file is accessible from our terminal
aws-ssh-connect2
Run the command in B to connect to the EC2 instance by SSH.
Once we run this command, the terminal will prompt us to accept the authenticity of the host. Type yes and press the Enter button to proceed.

Using SSH, we are now securely connected to our EC2 instance. This secure connection enables us to remotely manage and communicate with our server.

Download and Configure Git Action Runner
Git Action Runner acts as a link between our GitHub repository and the EC2 instance. This integration allows direct interaction between the two and enables automated build, test, and deployment processes.

To download and configure a Git Action Runner on our EC2 instance:

Go to the GitHub repository and click on Settings.
On the left-hand sidebar, click on Actions then select Runners.
In the Runners page click on the New self-hosted runner button.
git-action-runner
Here, we will choose the self-hosted runner image for our Ubuntu EC2 instance with the operating system set as Linux and architecture as x64.

Then step by step, run the following commands in the local SSH terminal:

git-action-runner2
Note: While running the command, it may prompt some setup questions, we can simply press Enter to skip to the default options.

After running the ./run.sh command, If the agent returns a ✅ Connected to GitHub message, it indicates a successful installation.

Next, we’ll install a service to run our runner agent in the background:

sudo ./svc.sh install
sudo ./svc.sh start
The above code will start our runner service in the background, making it ready to execute workflows whenever triggered.

Setting up a Node.Js Application Environment on an AWS EC2 Instance
We have successfully integrated our application on GitHub with the EC2 instance server using GitHub Actions Runner.

To ensure the smooth execution and operation of our Node.js application on the EC2 machine, we need to install essential libraries and dependencies for our application such as Node.js, NPM, and PM2.

To install NPM and Node.js, run the following command in the local SSH terminal:

sudo apt update
curl -sL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
sudo apt-get install -y nodejs
To install PM2, run the following command:

sudo npm install -g pm2
PM2 offers various useful commands for monitoring our application server. For more information about PM2, check out their documentation.

Our application is now fully set up and ready to go!

To start our application’s server, we need to navigate into the application’s folder in the EC2 instance.

To do this run the following command on the local SSH terminal

cd ~
cd /home/ubuntu/actions-runner/_work/cicd-app/cicd-app
Once we are inside the application’s folder, we can start the server in the background using pm2:

pm2 start src/index.js --name=backendserver
Using pm2 to start the server with a specified --name enables our Node.js server to be managed as a background service. This means our server will continue running even after we exit the SSH session. Additionally, pm2 provides continuous monitoring and ensures our application remains active and responsive at all times. This is very handy in production environments where we want our program to be available at all times.

Our Node.js application is now successfully up and running on the EC2 instance, and our CI/CD workflow has been configured.

The application will now be running and listening on the specified port 3000.

To ensure that the server is functioning correctly, we can easily check it through a web browser. Simply enter the server’s URL or IP address followed by the specified port.

For example, if our server’s IP address is 34.227.158.102, we would enter 34.227.158.102:3000 in the browser’s address bar.

If all configurations are correct, we’ll be greeted with the Products Page version 1.0 of our demo application.

live-cicd-app
Finally, we can proceed to test our CI/CD pipeline process. We will create an event that will act as a trigger to initiate a new workflow.

To do this, we will make a simple change to our HTML page. Specifically, we’ll update it from version 1 to version 2. Once this change has been made, we will push the updated code to the GitHub repository where our CI/CD workflow is defined. As soon as the push event is detected, our CI/CD pipeline will automatically kick off and execute the necessary steps to build, test, and deploy our updated application.

live-cicd-demo
Conclusion
By implementing this approach, we can automate our entire operation. CI/CD process, will improve our development workflow, making it efficient, reliable, and scalable. It enables us to confidently build new features, collaborate with the team, and deploy high-quality applications with more speed and ease
