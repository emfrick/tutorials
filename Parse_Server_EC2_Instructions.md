## Installing Parse Server on an EC2 RHEL Instance

Today, we will be walking through the steps to create a Parse server on Amazon Web Services EC2 cluster.  Since Parse is no longer providing hosting, we will need to setup our own Parse Server.

### High Level Steps
1. Launch an EC2 Instance
2. Setup EC2 Security Groups
3. Install NodeJS (6.X)
4. Install Parse Server

#### Launch an EC2 Instance

First, login to your AWS console and select the EC2 Compute link.  This will bring up the EC2 dashboard.  Select the *Launch Instance* button.  We're going to be setting up our Parse Server on the *Red Hat Free Tier Eligible* image, so click the *Select* button next to it.  You may choose any *Instance Type* you like, but I'm going to stick with the default free tier type.  Click *Configure Security Group* across the top and change the line for SSH *Source* to be *My IP*.  This will only allow access via SSH from your current computer.  Select *Review and Launch* and then *Launch*.

You will be presented with a popup to either select an existing key pair or create a new one.  Let's create a new one.  This will allow you to SSH into your AWS EC2 instance.  I will name mine "aws-parse-server" and then select *Download Key Pair*.  Save the .pem file somewhere that you can always have access to it.  Select *Launch Instance*. You will get a confirmation that your instance is launching.  Click on *View Instances* at the bottom of the page.

While we wait for our instance to finish initializing, let's move on to creating the necessary security groups.
>*Note:* You can set this up during the instance configuration if you wish.

#### Create the Security Groups
// TODO: Fill this out

#### Connect to your Instance
Now it's time to connect to our newly created EC2 instance.  Go to the *Instances* page, select the checkbox next to your new instance, and click *Connect*.  We will be connecting with a standalone SSH client.

Copy the key into your .ssh directory  
```bash
$ mv ~/aws-parse-server.pem ~/.ssh
```

Change the permissions  
```bash
$ cd ~/.ssh
$ chmod 400 aws-parse-server.pem
```

SSH to the Instance  
```bash
$ ssh -i "aws-parse-server.pem" ec2-user@<your_ec2_ip_address>
```

We're now ready to install NodeJS.

#### Installing NodeJS
Run these commands to ensure "sudo npm" can be run
```bash
sudo ln -s /usr/local/bin/node /usr/bin/node
sudo ln -s /usr/local/lib/node /usr/lib/node
sudo ln -s /usr/local/bin/npm /usr/bin/npm
sudo ln -s /usr/local/bin/node-waf /usr/bin/node-waf
```

#### Installing the Parse Server
Run Parse Server (pulled directly from [GitHub](https://github.com/ParsePlatform/parse-server))
```bash
$ npm install -g parse-server mongodb-runner
$ mongodb-runner start
$ parse-server --appId APPLICATION_ID --masterKey MASTER_KEY
```
> *Note*: I had to add the `--databaseURI mongodb://localhost/<dbname>` option to the `parse-server` command to get this to run.
