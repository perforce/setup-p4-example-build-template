# setup-p4-example-build-template

- [Summary](#summary)
  - [GitHub Actions Workflows Overview](#github-actions-workflows-overview)
    - [seed.yml](#seedyml)
    - [release.yml](#releaseyml)
- [Usage](#usage)

## Summary

This project is setup to be a demonstration of the `[setup-p4](https://github.com/perforce/setup-p4)` GitHub Action (login, client, sync, etc).  The project is based on a simple HTMT site that is located in this repository under the `base` directory.  

There are two GitHub Action Workflows stored in this repositry under `.github/workflows/` that will perform the following high level steps:



### GitHub Actions Workflows Overview

#### seed.yml

- checkout source code from this repository
- run p4 trust to trust your Helix Core
  - since this is the first P4 command being run it will initiate the download of the P4 CLI
- run p4 login
- Create a Helix Core workspace
- Add the `base` directory from this repostiry to a change list
- submit the change list to Helix Core



#### release.yml

* checkout github source code
* run p4 trust to trust your Helix Core
  - since this is the first P4 command being run it will initiate the download of the P4 CLI
* create workspace
* sync assets stored in helix core into GitHub Runner
* perform a build of the website
* publish the artifacts to GitHub Pages




## Usage

These are the steps to get this demo running in your own environment



---

**NOTE**

These steps assume that you have deployed Helix Core with the [Perforce Enhanced Studio Pack for Azure](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/perforce.perforce-enhanced-studio-pack). This demo is compatible with any Helix Core but some steps may need to be adjusted for your environment. 

---




- start by deploying a Helix Core for the purposes of this demo
  -  You can use the [Azure Enhanced Studio Pack](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/perforce.perforce-enhanced-studio-pack) to get a Helix Core up and running quickly
  -  NOTE: the helix core you use for this demo must be reachable by GitHub Actions

- Once the deployment is complete note the following outputs
   -  perforce username
   -  perforce password
   -  helix core public IP

- click the "Use this template" button to create your own repository

- select which ogranization you want the owner of of your new repository

- give your new repo a name

- select wether you want your project public or private 
  - unless you have a good reason you can keep the repo private

- Create Github secrets 
  - P4PORT
  - P4USER
  - P4PASSWD

- Now that the secretes are setup you can run the GitHub Workflow to seed data into Helix Core


  - In GitHub click on the Actions tab for your repository
  - On the left hand side click on Seed
  - Click on the "Run workflow"
  - Click the green "Run workflow" button leaving branch set at main

- Installing GitHub CLI
  - SSH into helix core instance

  - ```bash
    sudo su - perforce
    p4login -v 1
    sudo yum install -y https://github.com/cli/cli/releases/download/v2.5.2/gh_2.5.2_linux_amd64.rpm
    ```

  - authenticate the gh cli: 

    - ```bash
      echo "your github access token" | gh auth login --with-token
      ```

  - find your GitHub Action Workflow ID
    - ```bash
      /bin/gh workflow list --repo your github org/your github project name
      ```

    - For example:

      - ```bash
        /bin/gh workflow list --repo perforce/setup-p4-example-build-template
        ```

    - The output will look like the following:

      - ```
        [perforce@commit ~]$ /bin/gh workflow list --repo perforce/setup-p4-example-build-template
        Main           active  19232425
        ```

      - Note the ID for Main

  - Create a P4D Trigger to kick off GitHub Workflow whenever Content in Helix Core changes

    - ```bash
        p4 triggers -o > /tmp/triggers.spec
        
        cat <<EOF >> /tmp/triggers.spec
          github.commit push-commit /... "%quote%/bin/gh%quote% workflow run $GITHUB_WORKFLOW_ID --repo github org/repoistory name --ref main"
        EOF
        
        cat /tmp/triggers.spec | p4 triggers -i
        ```

    - For example:

        - ```bash
            p4 triggers -o > /tmp/triggers.spec
            
            cat <<EOF >> /tmp/triggers.spec
              github.commit push-commit /... "%quote%/bin/gh%quote% workflow run 19232425 --repo perforce/setup-p4-example-build-template --ref main"
            EOF
            
            cat /tmp/triggers.spec | p4 triggers -i
            ```

* After the pipeline has finished go to the repo settings and under the Pages section make the following changes
  * change the Source from None to gh-pages
  * set the folder to / (root) 
  * change the visibility from private to public
  * click save
* In a new browser tab open the following URL:
  *  https://aboutte.github.io/github-actions-demo-indie/
  *  change aboutte to your github username (or organization name if you forked into an organization)
* Now the environment is fully setup.  To test the end to end workflow lets make a change to content in Helix Core
  * to make a change you can use P4V
  * Manually kick off the workflow Edit











---
**NOTE**

After the pipeline finishes it takes about a minute for GitHub Pages to "see" the changes.  

---





