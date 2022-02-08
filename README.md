# helix-core-github demo (inspired by three-viewer)

- [helix-core-github demo (inspired by three-viewer)](#helix-core-github-demo-inspired-by-three-viewer)
  - [Usage](#usage)
    - [Demo Change](#demo-change)
  - [Source](#source)
    - [Todo](#todo)

This project is setup to be a demonstartion of the P4 GitHub Actions (login, client, sync, etc).  The project is based on Three Viewer but has been modified to store the models in Helix core and the web front end code in GitHub.  The pipeline stored in .github will perform the following high level steps:

* checkout github source code
* login into helix core
* create workspace
* sync assets stored in helix core into GitHub Runner
* perform a build of three viewer
* publish the artifact to GitHub Pages

The front end is pre setup to show a single asset from helix core but can be easily modified to show two assets from helix core.

## Usage

These are the steps to get this demo running in your own environment

* fork this project
* create the following GitHub Action Secrets (get the values for the secrest from Andy or Ryan)
  * P4PORT
  * P4USER
  * P4PASSWD
  * GH_PERSONAL_ACCESS_TOKEN
* Click on the Actions tab for your forked project and enable Actions
* For the "Build and Deploy" workflow click "Run workflow" to manually run the workflow using the workflow_dispatch for branch main
* After the pipeline has finished go to the repo settings and under the Pages section make the following changes
  * change the Source from None to gh-pages
  * set the folder to / (root) 
  * change the visibility from private to public
  * click save
* In a new browser tab open the following URL:
  *  https://aboutte.github.io/github-actions-demo-indie/
  *  change aboutte to your github username (or ogranization name if you forked into an orgaization)

### Demo Change

Out of the box the web front end will have three options

* Upload
* Default
* P4 asset Ship Dark

In a demo there is a quick and easy way to enable a 4th option.  Open src/index.vue and uncomment the div on line 31.  Commit and push this change to the main branch. This will trigger the workflow and publish a new version of the front end to Github Pages. 



## Source

* [P4 GitHub Actions](https://github.com/perforce/github-actions-p4)
* [Three Viewer](https://github.com/todaylg/three-viewer)


### Todo

- [x] Specular Glossiness Material

- [x] Panorama EnvMap(For the devices that dont support lod)

- [x] Fix energy loss in specular reflectance
  
- [x] Anisotropy(GGX)
  
- [x] Clearcoat

- [ ] Sheen

- [ ] Spot/Point Light and Shadow

- [ ] Post-processing(WIP)

- [ ] Ground Shadow

- [ ] Shadow Jitter
