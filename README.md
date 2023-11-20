
# await-remote-workflow
Trigger a workflow in another repo, and wait for it to complete successfully.

## How It Works
We define an optional ‘ci_uuid input for this workflow, which we’ll use when we remotely trigger this workflow. In the ‘jobs’ section, the first step is to write the full URL of this workflow to a temp file. The file name is the value of ‘ci_uuid. We’ll later query this URL in ‘workflow2’ and gather the exit status of ‘workflow1’. We then proceed to execute as many steps as we’d like, but wrap up the workflow with the ‘Upload UUID As An artifact' step. This would upload the temp file we created earlier on as an artifact of this workflow. Notice that this step would only run when 'ci_uuid is given as input, so scheduled runs (if any) or manual runs will not be affected.

## Usage

### In the "Caller" Workflow:
```
name: Build and Deploy A New Release
on:
  workflow_dispatch:
    inputs:
      release_string:
        description: Define the release

jobs:
  my-first-job:
    name: Create Release Tag
    runs-on: ubuntu-latest
    steps:
      - name: Do Something
        run: |
          echo "Do Stuff!"

  remote-await-job:
    name: Trigger Remote Workflow
    needs: [ my-first-job ]
    runs-on: ubuntu-latest
    steps:
      - name: Await Remote Job
        uses: Colby-Butler-Imprivata/await-remote-workflow@v1
        with:
          github-auth-token: ${{ secrets.MY_TOKEN }}
          token-type: 'Bearer'
          workflow-org: 'MY_ORG_NAME_HERE'
          workflow-repo: 'MY_REPO_NAME_HERE'
          workflow-yaml: 'MY_WORKFLOW_FILENAME.yml'
          workflow-branch: "MY_BRANCH_NAME"
          workflow-inputs: '{\"release_string\": \"Pass In Whatever Values are Expected By The Workflow you Are Calling. Yes, you must escape quotes because this was written in BASH\"}'
          wait-timeout-in-minutes: '10'

  my-next-job:
    name: Do this after the remote job has completed successfully
    needs: [ remote-await-job ]
    runs-on: ubuntu-latest
    steps:
      - name: Do Something
        run: |
          echo "Do Stuff!"
```
### In the Workflow You Want to Track:
```
name: Build and Deploy A New Release
on:
  workflow_dispatch:
    inputs:
      ci_uuid:
        description: 'Unique identifier used for pipeline tracking'
        type: string

jobs:
  create-tracking-artifact:
    if: ${{ inputs.ci_uuid }}
    runs-on: ubuntu-latest
    name: Create the Artifact To Track Workflow Completion
    steps:
      - name: Create Workflow Completion Tracking Artifact
        run: |
          echo $GITHUB_API_URL/repos/$GITHUB_REPOSITORY/actions/runs/$GITHUB_RUN_ID | tee /tmp/${{ inputs.ci_uuid }}

      - name: Upload Workflow Completion Tracking Artifact
        uses: actions/upload-artifact@v3
        with:
          name: ${{ inputs.ci_uuid }}
          path: /tmp/${{ inputs.ci_uuid }}

  my-last-job:
    name: The Last Job
    runs-on: ubuntu-latest
    steps:
      - name: Do Something
        run: |
          echo "Do Stuff!"

  trigger-workflow-completion-beacon:
      if: ${{ inputs.ci_uuid }}
      name: Trigger Beacon To Denote Workflow Completion After Your Last Job Completes Successfully
      needs: [ my-last-job ]
      runs-on: ubuntu-latest
      steps:
        - name: ${{ inputs.ci_uuid }}
          run: |
            echo "Job Complete. Beacon Deployed for UUID ${{ inputs.ci_uuid }}"

```
