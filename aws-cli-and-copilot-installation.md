## Install the latest version of the AWS CLI:

### Requirements:
- AWS CLI supports on Microsoft-supported versions of 64-bit Windows.
- Admin rights to install software.

### Install the AWS CLI:

1. Download and run the AWS CLI MSI installer for Windows (64-bit):

<https://awscli.amazonaws.com/AWSCLIV2.msi>

Alternatively, you can run the msiexec command to run the MSI installer.

```
C:\> msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```
For various parameters that can be used with msiexec, see msiexec on the Microsoft Docs website. For example, you can use the /qn flag for a silent installation.

```
C:\> msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi /qn
```

2. To confirm the installation, open the Start menu, search for cmd to open a command prompt window, and at the command prompt use the aws --version command.

```
C:\> aws --version
aws-cli/2.10.0 Python/3.11.2 Windows/10 exe/AMD64 prompt/off
```
If Windows is unable to find the program, you might need to close and reopen the command prompt window to refresh the path.

> Reference: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

## Configure AWS:

Connect with AWS administrator and get the Access Keys.

Open CMD. Run the following command to configure the AWS CLI with your AWS access key, secret key, default region, and output format:

```
C:\> aws configure
```
You'll be prompted to enter the following information, use the access key sheet provided by AWS administrator and enter the required information.

```
C:\> aws configure
AWS Access Key ID [None]: AKIAXU4DUDY5TZLCGS5G     #Enter your access key
AWS Secret Access Key [None]: fhAdeV4vEDKuDggsPPYj2K9qD/Ar71vZx5WI68S4     #Enter your secret key
Default region name: ap-south-1     #Enter the region your aws services are hosted
Default output format: json     #Enter output format if you know else leave empty
```
After configuring the AWS CLI, you can test it by running basic commands.

```
C:\> aws sts get-caller-identity
```

Remember that you might need to restart your Command Prompt or any other terminal windows after configuring the AWS CLI to ensure that the updated configuration is recognized.

> Reference: https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html

## Install the latest version of AWS Copilot

AWS Copilot is a tool to build, release, and operate production-ready containerized applications on Amazon ECS and AWS Fargate.

### Install Required Software:
Before installing AWS Copilot, you need to have a few prerequisites installed on your system:

- Docker: Make sure Docker is installed and running on your system.
- AWS CLI: Install the AWS Command Line Interface if you haven't already.

### Install Copilot:

Run the following command in PowerShell:

```
New-Item -Path 'C:\copilot' -ItemType directory; `Invoke-WebRequest -OutFile 'C:\copilot\copilot.exe' https://github.com/aws/copilot-cli/releases/latest/download/copilot-windows.exe
```

### Add in Environment Variable

- Open System Properties:
    - Press **Win + R** to open the **Run** dialog, type `sysdm.cpl`, and press Enter. This will open the **System Properties**.

- Click on **"Advanced"** tab in top bar.

- Click on the **"Environment Variables..."** button in the System Properties window.

- Under **"System variables"**, scroll down and find the **"Path"** variable. Select it and then click on the **"Edit..."** button.

- In the **"Edit Environment Variable"** window, click **"New"** Then, type or paste the path to the directory containing the Copilot executable. In our case, this would be `C:\copilot`.

- Click **"OK"** to close all the windows.

- Restart **Command Prompt** or **PowerShell** to apply the changes to the environment variables.

Now you should be able to run the Copilot executable (`copilot`) from any directory in the command prompt or PowerShell.

You can verify if Copilot is installed correctly by running:

```
copilot --version
```

> Now connect with the Project Solution Architect to know about existing Copilot projects and services deployed in your environment.

- **To initialize new Copilot Project:**
Navigate to a directory where you want to create your Copilot project and run:

```
copilot init
```

Follow the prompts to configure your project. This command will create a new application and environment.

- **Deploying Applications:**
Once your Copilot project is initialized, you can use Copilot to deploy and manage your applications.
