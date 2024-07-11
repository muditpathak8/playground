# Kosli Playground

A playground for learning how to implement Kosli.

This is a very simple demo application. It is made up of three components:

- WebApp: a single-page javascript web app.
- Alpha: a Ruby based API service
- Beta: a Ruby based API service

This repo is a monorepo with each component in its own directory with its own Makefile.
Each component has an independent GitHub Actions workflow. 
Each workflow will trigger when changes to the relevant component are pushed to the main branch.
Each workflow fakes the deployment step by doing a "docker compose up"


# Setting up

1. Fork this repo

2. Log into Kosli at https://app.kosli.com using Github
   This will create a Personal Kosli Organization whose name is your github username.

3. At https://app.kosli.com create a Docker Environment.
   This is the Kosli Environment that will record Snapshots of what is running in the "docker compose up" fake deployment.
   - Select Environments from left hand side menu
   - Click the blue `[Add new environment]` button at the top
   - Fill in the Name field as 'playground-prod'
   - Check the 'Docker host' radio button for the Type
   - Fill in the Description field
   - Leave "Exclude scaling events" checked
   - Leave "Compliance Calculation Off"
   - Click the blue `[Save environment]` button

4. Set the variables in the .env file
   - DOCKER_ORG_NAME to your Github username
   - REPO_NAME if you changed from playground
   - KOSLI_ORG to the name of your Kosli personal Org

5. Check you can build an image locally
   ```bash
   cd alpha
   make image
   cd ..
   ```
   This should create an image called: `ghcr.io/${DOCKER_ORG_NAME}/playground-alpha:0c74d4c`
   where `0c74d4c` will be the short-sha of your current HEAD commit.

6. Create a KOSLI_API_TOKEN secret.
   (Note: In a Shared Organization you would do this under a Service account) 
   - At https://app.kosli.com click your github user icon at the top-right
   - In the dropdown select `Profile`
   - Click the blue `[+ Add API Key]` button
   - Choose a value for the "API key expires in" or leave it as Never
   - Fill in the Description field
   - Click the blue `[Add]` button
   - You will see the api-key, something like `p1Qv8TggcjOG_UX-WImP3Y6LAf2VXPNN_p9-JtFuHr0`
   - Copy this api-key (Kosli stores an hashed version of this, so it will never be available again)
   - Set a Github Action secret, called KOSLI_API_TOKEN, set to the copied value


     
