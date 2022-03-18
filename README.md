# setup-p4-example-build-template

- [Summary](#summary)
  - [GitHub Actions Workflows Overview](#github-actions-workflows-overview)
    - [seed.yml](#seedyml)
    - [Deploy.yml](#deployyml)
- [Usage](#usage)
  - [Helix Core Deployment](#helix-core-deployment)
  - [GitHub Repository Setup](#github-repository-setup)
    - [Creating GitHub Secrets](#creating-github-secrets)
    - [Setup GitHub Pages](#setup-github-pages)
    - [Create GitHub Personal Access Token](#create-github-personal-access-token)
    - [Add Helix Core Trigger](#add-helix-core-trigger)
    - [Run GitHub Workflows](#run-github-workflows)
      - [Seed Data Into Helix Core](#seed-data-into-helix-core)
      - [Build and Deploy](#build-and-deploy)
      - [Edit](#edit)

## Summary

This project is setup to be a demonstration of the [setup-p4](https://github.com/perforce/setup-p4) GitHub Action.  The project is based on a simple HTML site that is located in this repository under the `base` directory.  

There are three GitHub Action Workflows stored in this repository under `.github/workflows/` that will perform the following high level steps:



### GitHub Actions Workflows Overview

#### seed.yml

- Checkout source code from this repository
- Run `p4 trust` to trust your Helix Core
- Login into the Helix Core
- Create a Helix Core workspace
- Add the `base` directory from this repository to a change list
- Submit the change list to Helix Core



#### Deploy.yml

* Create workspace
* Sync assets stored in helix core into GitHub Runner
* Perform a build of the website
* Publish the artifacts to GitHub Pages


## Workflow Diagram
![Workflow Diagram](/images/workflow.png)

## Usage

These are the steps to get this demo running in your own environment



---

**NOTE**

These steps assume that you have deployed Helix Core with the [Perforce Enhanced Studio Pack for Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/perforce.perforce-enhanced-studio-pack). This demo is compatible with any Helix Core but some steps may need to be adjusted for your environment. 

---



### Helix Core Deployment


- Start by deploying a Helix Core for the purposes of this demo
  -  You can use the [Azure Enhanced Studio Pack](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/perforce.perforce-enhanced-studio-pack) to get a Helix Core up and running quickly
  -  During the deployment configuration be sure to allow access from `0.0.0.0/0`.  More details can be found [here](https://github.com/perforce/setup-p4#network-connectivity).
- You will know when your deployment is complete when you see the green checkmark and the message "Your deployment is complete".  Note the following key outputs from the Outputs tab:
   -  `p4CommitPublicIP` - This is the IP address we will use to connect to Helix Core and will be used in the `P4PORT` secret later
   -  `helixCoreInstanceID` - This is the password for the Helix Core perforce user and will be used in the `P4PASSWD` secret later

### GitHub Repository Setup

Now that the Helix Core infrastructure is created it is time to create your GitHub Repository.  This repository, perforce/setup-p4-example-build-template, is setup as a [GitHub template repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-repository-from-a-template) to make it easy for you to create your own repository. 




- In this repository click the "Use this template" button to create your own repository
- Select which organization you want the owner of your new repository
- Give your new repo a name
- Set the repository to public, if you make your repository private GitHub Pages will not be available
- Check the box "Include all branches"
- Click "Create repository from template" button



#### Creating GitHub Secrets



With the repository created it is time to add [GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) to the repository.  These secrets will be used by the `perforce/setup-p4` action to connect to and authenticate to your Helix Core.  




- Open the Settings for your repository
- Click on Secrets and then Actions under Secrets
- Click on New repository secret
- Create Github Secrets for the following:

| GitHub Secret Name      | Value |  example|
| ----------- | ----------- | ----------- |
| P4PORT      | This will be in the form of `ssl:${p4CommitPublicIP}:1666` | If `p4CommitPublicIP` was 54.123.45.67 then your value would be `ssl:54.123.45.67:1666` |
| P4USER   		| perforce | perforce |
| P4PASSWD   	| Azure deployment `helixCoreInstanceID` output value | 6947879a-48f3-406a-9901-37ac597c968c |



#### Setup GitHub Pages

---
**NOTE**

Earlier when creating your repository from the template if you selected "public" these steps will be taken care of for you by a GitHub Bot.

---

* Open the Settings for your repository
* Click on Pages
* Verify the Source is set to gh-pages
* Verify the folder is set to / (root) 
* Click save if any changes were made
* In a new browser tab open the GitHub provided URL for your published site
* Keep this tab open as it will be used in future steps



#### Create GitHub Personal Access Token



For the GitHub CLI to be able to trigger a GitHub Action Workflow we will need to provide it a GitHub Personal Access Token. Follow the [GitHub documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) for creating a Personal Access Token.  We recommend creating a token with the following:

- Expire in 7 days 
- Has repo permissions
- Has workflow permissions
- and has read:org permissions (found under admin:org)


Take note of the Person Access Token, you will use it in a later step.


#### Add Helix Core Trigger



[Helix Core Triggers](https://www.perforce.com/manuals/p4sag/Content/P4SAG/chapter.scripting.triggers.html#:~:text=Helix%20server%20supports%20triggers%2C%20which,log%20in%20or%20change%20passwords.) are going to be used to initiate a GitHub Action Workflow every time a change list is submitted to Helix Core.  This will be accomplished by adding the [GitHub CLI](https://cli.github.com/) to your Helix Core instance. 


---

**NOTE**

Placeholder values that need to be filled in are surrounded$$ with `<<` and `>>`.  For example the `<<p4CommitPublicIP>>` in  `ssh centos@<<p4CommitPublicIP>>`  needs to be substituted for your public IP. 

---


- SSH to your Helix Core instance

  - `ssh centos@<<p4CommitPublicIP>>`
  - If `p4CommitPublicIP` was 54.123.45.67 then your command would be `ssh centos@54.123.45.67`

- Run the following commands to download and install the GitHub CLI

  - ```bash
    sudo su - perforce
    p4login -v 1
    sudo yum install -y https://github.com/cli/cli/releases/download/v2.5.2/gh_2.5.2_linux_amd64.rpm
    ```

- Authenticate the gh cli with the following command:

  - ```bash
    echo "<<GitHub Personal Access Token>>" | gh auth login --with-token
    ```

- Find your GitHub Action Workflow ID

  - ```bash
    /bin/gh workflow list --repo <<your-github-org>>/<<your-github-repository-name>>
    ```

  - For example if your org name is `acme` and you named your repository `setup-p4-demo` your command would be:

    - ```bash
      /bin/gh workflow list --repo acme/setup-p4-demo
      ```

  - The output will look like the following:

    - ```bash
      [perforce@commit ~]$ /bin/gh workflow list --repo acme/setup-p4-demo
      Build and Deploy           active  21567163
      Edit                       active  21567164
      Seed data into Helix Core  active  21567165
      ```
      
    - Note the ID for Build and Deploy

- Create a P4D Trigger to kick off your GitHub Workflow whenever content in Helix Core changes

---

**NOTE**

We recommend copying and paste the following bash snippet to a text editor to make your substitutions.  You can then copy and paste the whole snippet into your terminal.

---

  - ```bash
    p4 triggers -o > /tmp/triggers.spec
    
    cat <<EOF >> /tmp/triggers.spec
      github.commit change-commit /... "%quote%/bin/gh%quote% workflow run <<GITHUB_WORKFLOW_ID>> --repo <<your-github-org>>/<<your-github-repository-name>> --ref main"
    EOF
    
    cat /tmp/triggers.spec | p4 triggers -i
    ```

  - For example:

    - ```bash
      p4 triggers -o > /tmp/triggers.spec
      
      cat <<EOF >> /tmp/triggers.spec
        github.commit change-commit /... "%quote%/bin/gh%quote% workflow run 21567163 --repo acme/setup-p4-demo --ref main"
      EOF
      
      cat /tmp/triggers.spec | p4 triggers -i
      ```

* You can double check the trigger by running ` p4 triggers -o`.  If you need to make any edits just run `p4 triggers`



Now any time content is submitted to Helix Core the following command will be run:



```bash
/bin/gh workflow run 21567163 --repo acme/setup-p4-demo --ref main
```



which will run your Build and Deploy GitHub Action Workflow.



#### Run GitHub Workflows

---

**NOTE**

It is expected that the Build and Deploy workflow will have a failed run.  This run was triggered on repository creation which was before your GitHub Secrets were created.

---



##### Seed Data Into Helix Core


This workflow will take the `base` directory in your GitHub repository and commit it into Helix Core.  This workflow just needs to be run one time.


- In GitHub click on the Actions tab for your repository
- On the left hand side click on "Seed Data Into Helix Core
- Click on the "Run workflow"
- Click the green "Run workflow" button leaving branch set at main


---

**NOTE**

Now that the seed job has run, the source of truth for all assets is Helix Core and the assets have been removed from the main branch in GitHub.

---




##### Build and Deploy



This workflow will be triggered by Helix Core anytime changes are submitted to Helix Core. No need to manually trigger this one. To see this go back the Actions tab and select Build and Deploy.
You will see a running job that was initiated from the Helix Core Trigger. 



##### Edit



To simulate an end user making a change you have two options:



- Run the Edit Workflow which has an input parameter which will update the HTML background color
- Use P4V to edit and update the [HTML background color](https://github.com/perforce/setup-p4-example-build-template/blob/main/base/index.html#L12)



Either method will result in a change list being submitted to Helix Core which will trigger the Build and Deploy workflow. 



---
**NOTE**

After the Workflow finishes it takes about a minute for changes to GitHub Pages to be viewable by a browser.  

---
