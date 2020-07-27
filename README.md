# CircleCI Orb
[CircleCI](https://circleci.com/) is a leading continious integration and delivery platform for building workflow jobs. The xMatters orb gives CI/CD authors the ability to generate xMatters events as a command in a job as well as trigger an approval request. 

<kbd>
  <img src="https://github.com/xmatters/xMatters-Labs/raw/master/media/disclaimer.png">
</kbd>

# Pre-Requisites
* A CircleCI account
* xMatters account - If you don't have one, [get one](https://www.xmatters.com)!

# Files
* [CircleCI](CircleCI.zip) - The workflow with the form templates and inbound integration script. 

# How it works
There are two commands available in the [xMatters orb](https://circleci.com/orbs/registry/orb/xmatters/xmatters-orb). The first is `notify` and accepts individual parameters, most of which come from the [environment variables](https://circleci.com/docs/2.0/env-vars/#built-in-environment-variables). The other command is `notify_raw` and accepts a generic json payload. This allows infinite flexibility for the values in the payload which can then be acted on in the inbound integration script in xMatters. 
Each command will fire a webhook into an xMatters inbound integration script, which will then parse the payload to generate the event. 

# Installation

## xMatters set up

1. Upload the [CircleCI.zip](CircleCI.zip) workflow to the Workflows page.
2. Click on the workflow and navigate to the flows tab.
3. Click the link for **Build Notification** (or **Approval Request**) to display the flow designer canvas.
4. Double click on the **Inbound from CircleCI** step and click copy on the initiation url and save for later.


**Approval Requests**

If you are using the Approval Requests, then you will also need to set up the custom field for a personal access token. 

1. Open the Admin gear at the bottom of the navigation menu and click Custom Fields. (You will need to have the Company Supervisor role to see this page.)
2. Create a new custom field of type Text and a name of `CircleCI Token Plain`. It is important to use this exact name as it is referenced in the **Find User Property Value** step in the Approval Request canvas.
3. Save the field.

## CircleCI set up

1. To avoid making the xMatters inbound url publically accessible, it is recommended to store it in a [Project environment](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project) variable. Call this variable `XM_URL`. 

<kbd>
  <img src="media/XM_URL_proj_env.png" width="400">
</kbd>

2. Update your project `.circleci/config.yml` file to add a new step to execute either the `notify` or `notify_raw`. See the examples in the [orb](https://circleci.com/orbs/registry/orb/xmatters/xmatters-orb) for more details. 


**Approval Requests**

For Approval Requests, you will need a job type of `approval` in your workflow. Here is a simple example of a `config.yaml` file:

```
version: 2.1

orbs:
  xmatters: xmatters/xmatters-orb@1.0.4

jobs:
  build:
    docker: 
      - image: circleci/node:4.8.2
    steps:
      - run: echo "Building building building!"

  request-approval:
    docker:
      - image: circleci/node:4.8.2
    steps:
      - xmatters/notify:
          recipients: Engineering Managers

  hold:
    docker:
      - image: circleci/node:4.8.2
    steps:
      - run: echo "Hold for approval"

  deploy-stuff:
    docker:
      - image: circleci/node:4.8.2
    steps:
      - run: echo "Deploy stuff here"

workflows:
  build-test-and-approval-deploy:
    jobs:
      - build
      - request-approval
      - hold:
          type: approval
          requires:
            - build
            - request-approval
      - deploy-stuff:
          requires:
            - hold

```

Finally, you will need to generate a new [Personal Access Token](https://circleci.com/docs/2.0/managing-api-tokens/) and store that in your `CircleCI Token Plain` custom field. To access the custom field, navigate to your user profile record by clicking your username > Profile in xMatters. 

# Testing
Build the target project and inspect the build steps in the CircleCI UI and you should receive an event:

<kbd>
  <img src="media/CircleCi_Email.png" width="400">
</kbd>


# Troubleshooting
The CircleCi build log will have details up to the point it makes the curl request to xMatters:

<kbd>
  <img src="media/CircleCi_Build.png" width="400">
</kbd>

Any failure messages in making the call to xMatters will be displayed here. If this shows all successful, but there is still no xMatters event, check the Activity Stream in the inbound integration for any errors. 

