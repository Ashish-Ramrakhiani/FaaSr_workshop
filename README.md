# Objective

We will use this repository as a part of the hands-on activity for the FaaSr workshop at EFI 2025. 

This tutorial will guide you through the setup and execution of your first [FaaSr](https://faasr.io) workflow using an example of a simple forecast based on the README documentation of the [neon4cast package](https://github.com/eco4cast/neon4cast). The example performs the following steps as R functions that are composed into a FaaSr workflow deployable on GitHub Actions:

- Download and process target data to prep for model training and forecasts
- Generate forecast of variable A (temperature)
- Generation forecast of variable B (oxygen)
- Combine forecasts into a single file for submission to NEON Challenge

Through this activity, you will learn how to:

- Install the FaaSr package on an Rstudio environment
- Describe and configure a FaaSr workflow that combines R functions
- Execute the workflow in the cloud, using GitHub Actions
- Use a publically accessible S3 cloud storage using a Minio “bucket” for cloud data storage
- With the knowledge gained from this tutorial, you will be able to also run FaaSr workflows in OpenWhisk and Amazon Lambda, as well as use an S3-compliant bucket of your choice. 


# Pre-workshop requirements

Before we start running the workflow, you need to complete some prerequisites and keep the following things ready.
- A Github account
    - You most likely already have one, but in case you don't, you will need to create an account
- A Github Personal Access Token (PAT)
    - Create a short-lived GitHub PAT for this tutorial. [Detailed instructions to create a PAT are available here](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic); in summary:
  
    * In the upper-right corner of any page, click your profile photo, then click Settings.
    * In the left sidebar, scroll all the way to the bottom and click Developer settings.
    * In the left sidebar, click Personal access tokens (Classic).
    * Click Generate new token (Classic); *note: make sure to select Classic*
    * In the "Note" field, give your token a descriptive name.
    * In scopes, select “workflow” and “read:org” (under admin:org). 
    * Copy and paste the token; as you will need to save it to a file in your computer for use in the tutorial

- RStudio on Posit Cloud
    * [Click here](https://posit.cloud/) to open posit cloud in your browser window
    * Click on the "Learn more" button under the Free tier
    * Click on the Sign up button on the next page
    * Click on "Sign up with GitHub" button
    * Authorize posit cloud by clicking on the "Authorize-posit" button
    * Enter your github credentials when prompted
    * You should see the following screen once done: ![image](https://github.com/user-attachments/assets/ae6717ea-fe79-4c25-8451-d88b4ccef643)
- An S3 cloud storage bucket
     - We will provide you with credentials of a free temporary bucket during the workshop, so you don't need to do anything at this point
     
**You have now completed all the pre-workshop activities!**

***
# Workshop Day Activities

# Complete a pre-workshop survey
[Click here](https://forms.gle/GyCaMXR5Jjke4tr19) to access the pre-workshop survey. Complete "Section 1 pre-survey" and click on Next. Please keep this tab open as it will be needed to fill up a post-workshop survey at the end.

# Start your Rstudio session on Posit Cloud

[Login with your GitHub account](https://posit.cloud/login), and then [navigate to your Posit workspace](https://posit.cloud/content/yours?sort=name_asc). Click on New Project, and select "New RStudio Project". This will start RStudio and connect you to it through your browser.

# Configuring & deploying a FaaSr workflow.

## Clone the FaaSr workshop repo

First let's clone the FaaSr workshop repo - copy and paste this command in the R console:

```
system('git clone https://github.com/Ashish-Ramrakhiani/FaaSr_workshop.git')
```

In RStudio, Click on FaaSr_workshop folder on the lower right window (Files), then

Select More > Set as Working Directory from the drop down menu.

## Source the script that sets up FaaSr and dependences

Run following command, Fair warning: it will take a few minutes to install all dependences:

```
source('posit_setup_script')
```

## Configure FaaSr secrets file with your GitHub token

Open the file named "faasr_env" in a editor. You need to paste your GitHub token here: replace the string REPLACE_WITH_YOUR_GITHUB_TOKEN with your GitHub PAT, and save this file. 

This secrets file stores all the credentials we will use for this FaaSr workflow. You will notice that this file has some pre-populated credentials (secret key, access key) to access the Minio bucket.

## Configure the FaaSr JSON workflow

Head over to the files tab and open the neon_workflow.json file. This file stores the workflow configuration. Replace YOUR_GITHUB_USERNAME with your github username in the "UserName" property of the "ComputeServers" section. Next, update the "folder" parameter in all four functions by replacing "yourname" with your actual name (for example, change "FaaSr_workshop_yourname" to "FaaSr_workshop_jane"). This ensures your output files will be correctly stored in your personal directory on S3. A workflow has already been configured for you, shown in the image attached below. For additional information, please refer to [this schema](https://github.com/FaaSr/FaaSr-package/blob/main/schema/FaaSr.schema.json).

![image](https://github.com/user-attachments/assets/da1f0143-9387-431c-b466-818ddd943d67)


### Workflow Pipeline
1. Action `getData`
    - It is the starting point for our workflow that runs the `get_target_data` function 
    - It downloads and processes an aquatic dataset and uploads it to S3
    - On completion, it invokes the `oxygenForecastRandomWalk` and `temperatureForecastRandomWalk` actions
      
2. Actions `oxygenForecastRandomWalk` and `temperatureForecastRandomWalk`

    a. Action `oxygenForecastRandomWalk`
     - Retreives the processed dataset from S3 to create an oxygen forecast using the RandomWalk method and uploads it to S3
     - On completion, invokes the `combineForecastRandomWalk` action.
       
    b. Action `temperatureForecastRandomWalk`
     - Retreives the processed dataset from S3 to create a temperature forecast using the RandomWalk method and uploads it to S3
     - On completion, invokes the `combineForecastRandomWalk` action
       
 3. Action `combineForecastRandomWalk`
    - executes only when all predecessor actions are complete
    - Retreives the oxygen and temperature forecasts generated by previous actions, combines them and uploads the combined forecast to S3
    - This action marks the end of the workflow


## Register and invoke a simple workflow with GitHub Actions

Now you're ready for some Action! The below steps will:

* Use the faasr function from the FaaSr library to load neon_workflow.json and faasr_env in a list called faasr_workshop
* Use the register_workflow() function to create a repository called FaaSr_workshop_actions in GitHub, and configure the workflow using GitHub Actions
* Use the invoke_workflow() function to invoke the execution of your workflow

Paste the following commands to your console:

```
faasr_workshop<- faasr(json_path="neon_workflow.json", env="faasr_env")
faasr_workshop$register_workflow()
```

When prompted, select "public" to create a public repository. Now to invoke the workflow, run the following command:

```
faasr_workshop$invoke_workflow()
```

Head over to the GitHub repository `FaaSr_workshop_actions` that was just created by FaaSr in your GitHub profile. Click on the "Actions" tab in the repository navigation bar to monitor your workflow execution status.
You should see a list of workflow runs with their status indicators. Look for completed runs with green checkmarks, which indicate successful execution of all steps in your forecast workflow.
The screenshot below shows how a successful workflow execution should appear:

![Screenshot from 2025-04-23 13-39-28](https://github.com/user-attachments/assets/75c645ef-ab49-46b7-b96a-80e541d697a2)

This simple workflow you just executed consists of four R functions: 

1. `get_target_data.R` - Downloads and processes the aquatic dataset, creating two CSV files:
   - `aquatic_full.csv`: The complete dataset organized by datetime and site ID
   - `blinded_aquatic.csv`: A training dataset that excludes the most recent 35 days, used for forecast development

2. `create_oxygen_forecast_rw.R` - Generates oxygen forecasts saved as `oxygen_fc_rw.csv`

3. `create_temperature_forecast_rw.R` - Generates temperature forecasts saved as `temperature_fc_rw.csv`

4. `combine_forecasts_rw.R` - Merges the oxygen and temperature forecasts into a single output file: `rw_forecast_combined.csv`

## Browse the S3 Data Store to view outputs

### Using Minio console


When all the runs are successful, you can explore the forecast files generated in Minio by visiting https://play.min.io:9443. Log in with the following credentials:
```
Username: Q3AM3UQ867SPQQA43P2F
Password: zuf+tfteSlswRu7BJ86wekitnifILbZam1KYY3TG
```

Search for the bucket named "faasr" and look for the "FaaSr_workshop_yourname" folder in the faasr bucket, you should be able to see the forecast files created by the workflow.

### Using Minio client

You can also use the mc_ls command to browse the outputs in the console:
Remember to replace "yourname" with your actual name in these commands, just as you did in the configuration file.

```
mc_ls("play/faasr/FaaSr_workshop_yourname")
mc_cat("play/faasr/FaaSr_workshop_yourname/blinded_aquatic.csv")
mc_cat("play/faasr/FaaSr_workshop_yourname/oxygen_fc_rw.csv")
mc_cat("play/faasr/FaaSr_workshop_yourname/temperature_fc_rw.csv")
mc_cat("play/faasr/FaaSr_workshop_yourname/rw_forecast_combined.csv")

```
# Complete a post-workshop survey
Switch to the [survey](https://forms.gle/GyCaMXR5Jjke4tr19) tab again and complete "Section 2 post-survey". Thank you for attending the FaaSr workshop.

# Using the FaaSr workflow builder

While you can create and edit FaaSr configuration files in any text editor, FaaSr also provides a Shiny app graphical user interface to facilitate the development of simple workflows using your browser. You can use it to edit a configuration from scratch, or from an existing JSON configuration file that you upload as a starting point, and then download the edited JSON file to your computer for use in FaaSr, for example as in faasr_workshop <- faasr(json_path="neon_workflow.json", env="faasr_env")

[Right-click here to open the FaaSr workflow builder in another window](https://faasr.shinyapps.io/faasr-json-builder/). To test it, you can upload the neon_workflow.json from this tutorial, and you will be able to visualize and edit the workflow shown above.
