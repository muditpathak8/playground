
# Kosli Playground

A playground for learning how to implement [Kosli](https://kosli.com)

This is a very simple demo application. It is made up of three components:

- WebApp: a single-page javascript web app.
- Alpha: a Ruby based API service
- Beta: a Ruby based API service

This repo is a monorepo with each component in its own directory with its own Makefile.
Each component has an independent GitHub Actions workflow. 
Each workflow will trigger when changes to the relevant component are pushed to the main branch.


# Setting up

## [Fork this repo](https://github.com/kosli-dev/playground/fork)

Follow the remaining instructions in this README from your forked repo.


## Log into Kosli at https://app.kosli.com using GitHub

Logging in using GitHub creates a Personal Kosli Organization whose name is your GitHub username.
You cannot invite other people to your personal organization; it is intended only to try Kosli out
as you are now doing. For real use you would create a Shared Kosli Organization (from the top-right 
dropdown next to your user-icon) and invite people to it.


## At https://app.kosli.com create a Docker Environment

In this playground the CI pipelines fake their deployment step by doing a "docker compose up".
Create a Kosli Environment to record what is running in this fake deployment.
- Select `Environments` from left hand side menu
- Click the blue `[Add new environment]` button at the top
- Fill in the Name field as `playground-prod`
- Check the `Docker host` radio button for the Type
- Fill in the `Description` field
- Leave `Exclude scaling events` checked
- Leave `Compliance Calculation Require artifacts to have provenance` set to `Off`
- Click the blue `[Save environment]` button
- Open a tab in your browser for the `playground-prod` Kosli Environment as we will often review how it changes 


## Set the .env file variables

- Edit the [.env](.env) file as follows:
  - KOSLI_ORG to the name of your Kosli personal Org
  - DOCKER_ORG_NAME to your GitHub username in lowercase
  - REPO_NAME if you changed it from `playground`


## Check you can build and run an image locally [optional]

```bash
make -C alpha image
```
This should create an image called: `ghcr.io/${DOCKER_ORG_NAME}/${REPO_NAME}-alpha:0c74d4c`
where `0c74d4c` will be the short-sha of your current HEAD commit.
```bash
make -C alpha run
```
This should run the image locally, in a container, on port 4500.
Check you can reach `localhost:4500` in your browser.
It should show the string `Alpha` and nothing else.


## Create a KOSLI_API_TOKEN and save it as a GitHub Action secret

(Note: In a Shared Organization you would do this under a Service account) 
- At https://app.kosli.com click your GitHub user icon at the top-right
- In the dropdown select `Profile`
- Click the blue `[+ Add API Key]` button
- Choose a value for the `API key expires in` or leave it as Never
- Fill in the `Description` field
- Click the blue `[Add]` button
- You will see the api-key, something like `p1Qv8TggcjOG_UX-WImP3Y6LAf2VXPNN_p9-JtFuHr0`
- Copy this api-key (Kosli stores a hashed version of this, so it will never be available from https://app.kosli.com again)
- Create a GitHub Action secret (at the repo level), called `KOSLI_API_TOKEN`, set to the copied value


# Understand the fake deployment in the CI pipeline

- The repo is set up as a monorepo, with dirs called `alpha`, `beta`, and `webapp`
  for the three services. The `.github/workflows` files have `on: paths:` filters set, so they only run when
  there is a change in their respective directory (or the workflow file itself)
- There is a "FAKE" [deploy](.github/workflows/alpha_main.yml#L128) job which runs this command:
  ```yml
  docker compose up ${{ env.SERVICE_NAME }} --wait
  ```
  After this command, the CI pipeline installs the Kosli CLI, and then runs this command:
  ```yml
  kosli snapshot docker "${KOSLI_ENVIRONMENT_NAME}"
  ```
  The [kosli snapshot docker](https://docs.kosli.com/client_reference/kosli_snapshot_docker/) command takes a snapshot 
  of the docker containers currently running (inside the CI pipeline)
  and sends their image names and digests/fingerprints to the named Kosli Environment (`playground-prod`).
  This command does _not_ need to set the `--org`, or `--api-token` flags because
  the `KOSLI_ORG` and `KOSLI_API_TOKEN` environment variables have been set at the top of the workflow yml file.


## Make a change, run the CI workflow, review the Environment in Kosli

- Edit the file [alpha/code/alpha.rb](alpha/code/alpha.rb) so the return string from the `'/'` route is a new string
- Commit (add+commit+push if not editing in GitHub)
- Wait for the GitHub Action Workflow to complete.
- Refresh the `playground-prod` Environment at https://app.kosli.com and verify it shows the `playground-alpha` 
image running. The image tag should be the short-sha of your new HEAD commit 
- This playground-alpha Artifact currently has no [provenance](https://www.kosli.com/blog/how-to-secure-your-software-supply-chain-with-artifact-binary-provenance/
) but is nevertheless showing as Compliant. This is because the Environment was set up with `Require artifacts to have provenance`=Off. 
We will provide provenance shortly.


## Make another change, rerun the CI workflow, review the Environment in Kosli

- Re-edit the file [alpha/code/alpha.rb](alpha/code/alpha.rb) so the return string from the `'/'` route is a new string
- Commit (add+commit+push if not editing in GitHub)
- Wait for the GitHub Action Workflow to complete
- Refresh the `playground-prod` Environment at https://app.kosli.com and in the [Log] view verify
  - the previous playground-alpha Artifact has exited
  - the new playground-alpha Artifact is running, and this Artifact has provenance (there is a commit short-sha and a commit message)


## Create a Kosli Flow and Trail

- Kosli attestations must be made against a Trail, living inside a Flow.
  - A Kosli Flow represents a business or software process for which you want to track changes and monitor compliance.
    You create a Kosli Flow with the [kosli create flow](https://docs.kosli.com/client_reference/kosli_create_flow/) command.
  - A Kosli Trail represents a single execution instance of a process represented by a Kosli Flow. 
    Each trail must have a unique identifier of your choice, based on your process and domain. 
    Example identifiers include git commits or pull request numbers.
    You begin a Kosli Trail with the [kosli begin trail](https://docs.kosli.com/client_reference/kosli_begin_trail/) command.
- At the top of the [.github/workflows/alpha_main.yml](.github/workflows/alpha_main.yml) file add two new `env:` variables for the
Kosli Flow (named after your repo) and Kosli Trail (named after each git-commit), as follows:
```yml
env:
  KOSLI_FLOW: playground-alpha-ci
  KOSLI_TRAIL: ${{ github.sha }}
```
- Still in [.github/workflows/alpha_main.yml](.github/workflows/alpha_main.yml), add the following entries to the end of the `setup:` job
to install the Kosli CLI and create the Kosli Flow and Kosli Trail.
```yml
      - uses: actions/checkout@v4.1.1

      - name: Setup the Kosli CLI
        uses: kosli-dev/setup-cli-action@v2
        with:
          version: ${{ vars.KOSLI_CLI_VERSION }}

      - name: Create the Kosli Flow for this pipeline
        run:
          kosli create flow "${{ env.KOSLI_FLOW }}"
            --description="Learning about Kosli"
            --use-empty-template

      - name: Begin Kosli Trail for this commit
        run:
          kosli begin trail "${{ env.KOSLI_TRAIL }}"
            --description="${{ github.actor }} - $(git log -1 --pretty=%B)"
```
- Commit (add+commit+push if not editing in GitHub)
- Wait for the GitHub Action Workflow to complete
- In https://app.kosli.com, click `Flows` on the left hand side menu
- Click the Flow named `playground-alpha-ci`
- You should see a single Trail whose name is the repo's current HEAD commit
- Click the Trail name to view it, and confirm this Trail has no attestations
- Is there a new Snapshot in the `playground-prod` Environment?


## Attest the provenance of the Artifact in the CI pipeline

- Most attestations need the Docker image digest/fingerprint. We will start by making this available to all jobs.
- In [.github/workflows/alpha_main.yml](.github/workflows/alpha_main.yml)...
  - uncomment the following comments near the top of the `build-image:` job
  ```yml
  #    outputs:
  #      artifact_digest: ${{ steps.variables.outputs.artifact_digest }}
  ```
  - uncomment the following comments at the bottom of the `build-image:` job
  ```yml
  #    - name: Make image digest available to following jobs
  #      id: variables
  #      run: |
  #        DIGEST=$(echo ${{ steps.docker_build.outputs.digest }} | sed 's/.*://')
  #        echo "artifact_digest=${DIGEST}" >> ${GITHUB_OUTPUT}
  ```
  - add the following to the end of the `build-image:` job
  to install the Kosli CLI and attest the Artifact's digest/fingerprint.
  ```yml
      - name: Setup Kosli CLI
        uses: kosli-dev/setup-cli-action@v2
        with:
          version: ${{ vars.KOSLI_CLI_VERSION }}

      - name: Attest image provenance to Kosli Trail
        run: 
          kosli attest artifact "${needs.setup.outputs.image_name}" 
            --artifact-type=docker
            --name=alpha
  ```
- Note that `kosli attest` commands do not need to specify the `--org` or `--flow` or `--trail` flags because there are 
environment variables called `KOSLI_ORG`, `KOSLI_FLOW`, and `KOSLI_TRAIL`.
  - In the [kosli attest artifact](https://docs.kosli.com/client_reference/kosli_attest_artifact/) command above, the 
  Kosli CLI calculates the fingerprint. To do this the CLI needs to be told
  the name of the Docker image (`${needs.setup.outputs.image_name}`), and that this is a Docker image
  (`--artifact-type=docker`), and that the image has previously been pushed to its registry (which it has)
  - You can also provide the fingerprint directly using the `--fingerprint` flag or `KOSLI_FINGERPRINT` environment 
    variable. We will see an example of this shortly.
- Commit (add+commit+push if not editing in GitHub)
- Wait for the GitHub Action Workflow to complete
- Refresh the `playground-prod` Environment at https://app.kosli.com 
- You will see a new Snapshot
- The Artifact will have Provenance


## View a deployment diff

- Re-edit the file [alpha/code/alpha.rb](alpha/code/alpha.rb) so the return string from the `'/'` route is a new string
- Commit (add+commit+push if not editing in GitHub)
- Wait for the GitHub Action Workflow to complete
- Refresh the `playground-prod` Environment at https://app.kosli.com
- You will see a new Snapshot
- Its Artifact will have Provenance
- Click the `>` chevron to reveal more information in a drop-down
- Click the link titled `View diff` in the entry called `Previous` to see the deployment-diff; the commit-level diff 
between the currently running alpha Artifact, and the previously running alpha Artifact.


## Attest unit-test evidence to Kosli

- [.github/workflows/alpha_main.yml](.github/workflows/alpha_main.yml) has a `unit-test:` job. You will attest its results to Kosli
- Add the following to the end of the `unit-test:` job to install the Kosli CLI, and attest the unit-test results
```yml
    - name: Install the Kosli CLI
      uses: actions/checkout@v4.1.1
      with:
        fetch-depth: 1
          
    - name: Attest unit-test results to Kosli
      env:
        KOSLI_FINGERPRINT: ${{ needs.build-image.outputs.artifact_digest }}
      run:
        kosli attest junit --name=alpha.unit-test --results-dir=alpha/test/reports/junit
```
- Commit (add+commit+push if not editing in GitHub)
- Wait for the GitHub Action Workflow to complete
- Refresh the `playground-prod` Environment at https://app.kosli.com and verify it shows the new `playground-alpha` 
image running. The image tag should be the short-sha of your new HEAD commit
- 

