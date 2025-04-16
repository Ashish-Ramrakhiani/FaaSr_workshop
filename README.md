# Objective

We will use this repository as a part of hands on activity for the FaaSr workshop. This tutorial will help you through the setup and execution of a FaaSr workflow using [neon4cast package](https://github.com/eco4cast/neon4cast). You will learn how to describe, configure, and execute a FaaSr workflow of R functions in the cloud, using GitHub Actions for cloud execution of functions, and a public Minio S3 “bucket” for cloud data storage. With the knowledge gained from this tutorial, you will be able to also run FaaSr workflows in OpenWhisk and Amazon Lambda, as well as use an S3-compliant bucket of your choice. 


# Pre-workshop requirements

Before we start running the workflow, you need to complete some prerequisites and keep the following things ready.
- A Github account
- A Github Personal Access Token (PAT)
    - Create a short-lived GitHub PAT for this tutorial. [Detailed instructions to create a PAT are available here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic); in summary:
  
    * In the upper-right corner of any page, click your profile photo, then click Settings.
    * In the left sidebar, click Developer settings.
    * In the left sidebar, click Personal access tokens (Classic).
    * Click Generate new token (Classic); *note: make sure to select Classic*
    * In the "Note" field, give your token a descriptive name.
    * In scopes, select “workflow” and “read:org” (under admin:org). 
    * Copy and paste the token; as you will need to save it to a file in your computer for use in the tutorial
- a Minio S3 bucket (you can use the [Play Console](https://min.io/docs/minio/linux/administration/minio-console.html#minio-console) to use a public/unauthenticated server)
- RStudio on Posit Cloud
    * [Click here](https://posit.cloud/) to open posit cloud in your browser window
    * Click on the "Learn more" button under the Free tier
    * Click on the Sign up button on the next page
    * Click on "Sign up with GitHub" button
    * Authorize posit cloud by clicking on the "Authorize-posit" button
    * Enter your github credentials when prompted
    * You should see the following screen once done: ![image](https://github.com/user-attachments/assets/ae6717ea-fe79-4c25-8451-d88b4ccef643)

# Start your Rstudio session on Posit Cloud

[Login with your GitHub account](https://posit.cloud/login), and then [navigate to your Posit workspace](https://posit.cloud/content/yours?sort=name_asc). Click on New Project, and select "New RStudio Project". This will start RStudio and connect you to it through your browser.

# Configuring & deploying a FaaSr workflow.

## Clone the FaaSr workshop repo

First let's clone the FaaSr workshop repo - copy and paste this command in the R console:

```
system('git clone https://github.com/Ashish-Ramrakhiani/FaaSr_workshop.git')
```

In RStudio, navigate to the FaaSr_workshop folder, then set it as the working directory (More > Set as Working Directory).



## Source the script that sets up FaaSr and dependences

Run following command, Fair warning: it will take a few minutes to install all dependences:

```
source('posit_setup_script')
```
## Configure Rstudio to use GitHub Token

Within Rstudio, configure the environment to use your GitHub account (replace with your username and email). Paste the following in the console:

```
usethis::use_git_config(user.name = "YOUR_GITHUB_USERNAME", user.email = "YOUR_GITHUB_EMAIL")
```

Now set your GitHub token as a credential for use with Rstudio - paste your token in the pop-up window that opens up when you run the following in your console:

```
credentials::set_github_pat()
```

## Configure the FaaSr secrets file with your GitHub token

Open the file named "faasr_env" in a editor. You need to paste your GitHub token here: replace the string "REPLACE_WITH_YOUR_GITHUB_TOKEN" with your GitHub PAT, and save this file. 

This secrets file stores all the credentials we will use for this FaaSr workflow. You will notice that this file has some pre-populated credentials (secret key, access key) to access the Minio "play" bucket.

## Configure the FaaSr JSON workflow

Head over to files tab and open the `neon_workflow.json` file. This file stores the workflow configuration. Replace "YOUR_GITHUB_USERNAME" with your github username in the username of ComputeServer section. A workflow has already been configured for you, shown in the image attached below. For additional information, please refer to [this schema](https://github.com/FaaSr/FaaSr-package/blob/main/schema/FaaSr.schema.json).

![image](https://github.com/user-attachments/assets/da1f0143-9387-431c-b466-818ddd943d67)


### Workflow Pipeline
1. getData - Invoke function:
    - Downloads and processes an aquatic dataset and uploads it to S3
    - This function is the starting point for our workflow, which simultaneously invokes oxygenForecastRandomWalk and temperatureForecastRandomWalk next.
2. The following two functions will run simultaneously:

   a. oxygenForecastRandomWalk
     - Uses the processed dataset to create oxygen forecast using the RandomWalk method and uploads it to S3
     - Invokes combined forecast next.
       
    b.temperatureForecastRandomWalk
     - Uses the processed dataset to create temperature forecast using RandomWalk method and uploads it to S3
     - Invokes combined forecast next.
 4. combineForecastRandomWalk:
    - Downloads the generated forecast files from S3, combines them to store the final combined forecast on S3
    - This function marks the end of the workflow


## Register and invoke the simple workflow with GitHub Actions

Now you're ready for some Action! The steps below will:

* Use the faasr function in the FaaSr library to load the neon_workflow.json and faasr_env in a list called neon4cast_tutorial
* Use the register_workflow() function to create a repository called neon4cast_faasr_actions in GitHub, and configure the workflow there using GitHub Actions
* Use the invoke_workflow() function to invoke the execution of your workflow

Paste the following commands to your console:

```
neon4cast_tutorial<- faasr(json_path="neon_workflow.json", env="faasr_env")
neon4cast_tutorial$register_workflow()
```

When prompted, select "public" to create a public repository. Now to invoke the workflow, run the following command:

```
neon4cast_tutorial$invoke_workflow()
```

## Check if action is successful

Head over to the github repo `neon4cast_faasr_actions` just created by FaaSr, go to actions page to see if your actions has successfully run. 
If the runs are successful, you can explore the Console using https://play.min.io:9443. Log in with the following credentials:
```
Username: Q3AM3UQ867SPQQA43P2F
Password: zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
```

Look for the neon4cast folder in faasr bucket, you should be able to see the forecasts you have just created.


