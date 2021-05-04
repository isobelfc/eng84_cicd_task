# CICD

## CICD Basics

### What is CICD
- continuous integration, continuous delivery/deployment
- CI/CD Pipeline
  - commit
  - build
  - automate tests
  - deploy

### Available Tools
- Jenkins
- Ansible
- AWS
- Trello

### Continuous Delivery vs Continuous Deployment
- in continuous delivery release of software is manual
- in continuous deployment release is automated

## Task
- merge CI from dev to main if the tests passed
- deploy on AWS EC2 once the code is merged to main

## Process

### Job 1 - CI
- create a new branch called `dev` on the Sparta app repository
- create a new SSH key and add it to the settings of this repository
- create a new Jenkins job
- set the repository URL to the SSH link to the repository, and the credentials to be the new key you added
- set branch specifier as `*/dev`
- set the build trigger to be `GitHub hook trigger for GITScm polling`
- this will cause the job to run when changes are pushed to the Github repository
- select `Execute shell` under `Build` and insert the following code:
```
cd app
npm install
npm test
```
- this will check whether the app homepage loads correctly and includes the word `Sparta`, and whether the Fibonacci page loads and displays the correct value for the 10th number in the sequence

### Job 2 - Merge
- set the repository URL to the same as the last job
- under `Build Triggers`, set it to build after other projects are built, and only run if the previous project is stable
- under `Git Publisher`, tick `Push Only If Build Succeeds`, and for `Branches` select `main` as branch to push and `origin` as target remote name
- set `SSH Agent` to the key you set up before

### Job 3 - Deploy
- set to build only if the merge build is stable
- set the SSH Agent to `DevOpsStudent`
- insert the following under `Execute Shell`:
```
rm -rf eng84_cicd_jenkins*
git clone -b main https://github_repository_url.git
cd eng84_cicd_jenkins
rsync -avz -e "ssh -o StrictHostKeyChecking=no" app ubuntu@ip:/home/ubuntu/app
rsync -avz -e "ssh -o StrictHostKeyChecking=no" environment ubuntu@ip:/home/ubuntu/app/environment

ssh -o "StrictHostKeyChecking=no" ubuntu@ip <<EOF

  killall npm
  cd app/app
  sudo npm install
  nodejs seeds/seed.js
  
  nodejs app.js &
  
EOF
```
- the `rm -rf` command removes the existing folder on the drone EC2, and all its contents
- `git clone` then clones the updated version of the code from the repository to replace it
- we then cd into the new folder and use `rsync` to copy the `app` and `environment` folders into the EC2 instance we are running the app on
- `ssh -o "StrictHostKeyChecking=no"` allows us to bypass the confirmation prompt we would get otherwise
- we then SSH into the app instance, again bypassing the confirmation
- inside the app instance, we kill any running processes with `killall npm`
- we then cd into the app folder, install npm, and run `seed.js` to seed the posts database
- we can then run `app.js` to get our updated app up and running
- if we have configured this correctly, when we push code to the dev branch, it should automatically trigger a series of tests, and then merge the dev branch into main and relaunch the app with the updated code if the tests pass