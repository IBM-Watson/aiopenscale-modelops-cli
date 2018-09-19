---

copyright:
  years: 2018
lastupdated: "2018-09-17"

---

# Contents of this file

* Introduction
* Requirements
* Installation
* Configuration
* Integration of IBM AI OpenScale Model Operation Task CLI with CI/CD Tools
* Troubleshooting
* FAQ

## Introduction

The IBM AI OpenScale CLI model operations tool allows you to execute tasks related to the lifecycle management of machine learning models. This tool is complementary to the IBM Cloud CLI tool, augmented with the machine-learning plugin: [https://www.ibm.com/support/knowledgecenter/DSXDOC/analyze-data/ml_dlaas_environment.html](https://www.ibm.com/support/knowledgecenter/DSXDOC/analyze-data/ml_dlaas_environment.html)

These tools can typically be used in conjunction with a CI/CD system, such as Jenkins or Travis CI, but can also be invoked in a stand-alone fashion.

Using the IBM AI OpenScale CLI model operations tool, you can execute the following tasks:

* `uploadData` - Upload local data to IBM Cloud Object Storage
* `runDataFlow` - Run a given data flow
* `syncData` - Creates a copy of an object from a IBM Cloud Object Storage Bucket into a target IBM Cloud Object Storage Bucket
* `deployCF` - Deploys application to Cloud Foundry
* `postSlack` - Post a message to a Slack channel
* `postEmail` - Send email to a specified recipient emails.
* `triggerTravis` - To trigger Travis CI build
* `triggerJenkins` - To trigger Jenkins build

For general help with the tool, invoke: `modelops.sh -h`.

For help with a specific task, including documentation for task options, invoke: `modelops.sh task-id -h`, where `task-id` is one of the tasks listed above.

## Requirements

This tool requires a Java JRE/JDK at version 8 or higher. The `java` command must be accessible in the environment where this tool is installed.

In order to invoke machine learning model-specific tasks, such as training, deploying, or scoring a model, you should also install the [IBM Cloud CLI tool](https://console.bluemix.net/docs/cli/reference/ibmcloud/download_cli.html#install_use), along with the [machine-learning plugin](https://dataplatform.cloud.ibm.com/docs/content/analyze-data/ml_dlaas_environment.html?audience=wdp&context=wdp#step2).

## Installation

Extract the `aiopenscale-modelops-cli file` (`.tar.gz`) into a suitable directory. Once extracted, you should see the following files:

* `modelops.sh` - the main entry point used to run a task
* `uploadData.properties` - a task input template for the uploadData task
* `run_dataflow.properties` - a task input template for the runDataFlow task
* `syncData.properties` - a task input template for the syncData task
* `deployCF.properties` - a task input template for the deployCF task
* `post_slack.properties` - a task input template for the postSlack task
* `post_email.properties` - a task input template for the postEmail task
* `triggerJenkins.properties` - a task input template for the triggerJenkins task
* `triggerTravis.properties` - a task input template for the triggerTravis task
* `model-operation-tasks-cli.jar` - a library required to run tasks
* `README.md` - this document

## Configuration

Each invocation of the `modelops.sh` entry point can invoke one task. If you need to set the arguments using environment variables, you can use
environment variable place holders of the form `${ENV_VAR_NAME}`, where ENV_VAR_NAME can be any suitable environment variable name.  This may be useful in situations where you need to dynamically set the argument value, or need to avoid using hard-coded sensitive information.

There are two ways you can invoke a task:

1.  Invoke the task using a `.properties` file

    In order to run the task, copy one of the included task properties files for the task you intend to run, and then modify it to suit your task execution. You may find this approach suitable for situations where you are interactively invoking tasks and you do not want to repeatedly supply task arguments on the command-line.

    As an example, execute a task to post a message to Slack, using a `.properties` file:

    * Copy the `post_slack.properties`, for example: `cp post_slack.properties my_post_slack.properties`
    * Edit your copy of the `.properties` file to have appropriate values, for example:

      `channel=#modelops-alerts`

      `text=Finished training model: Test Model 1`

      `url=${MODELOPS_SLACK_INCOMING_WEBHOOK_URL}`

      **Note**: you can change to use any suitable environment variable name

    * Each task execution requires its own properties file. If you require additional task execution configurations, repeat the above steps.
    * Use the `modelops.sh` entry point to execute the task, for example: `./modelops.sh --file my_post_slack.properties`
    * See the `invoke_multiple_tasks.sh` script, to see how you can write your own scripts to execute a series of tasks conditionally.

1.  Invoke the task using command-line arguments

    In order to run the task using command-line arguments, you can specify the same properties you see in the `.properties` file preceded by two dashes (`--`), followed by a space and then the argument value.  Be sure to use single or double quotes as appropriate, if the value contains any special characters.

    You may find this approach suitable for situations where you need to dynamically set the task arguments based on values that you calculate in a job, such as a Jenkins or Travis CI job.

    As an example, execute a task to post a message to Slack, using command-line arguments:

    `./modelops.sh postSlack --channel '#modelops-alerts' --text 'Finished training model: Test Model 1' --url '${MODELOPS_SLACK_INCOMING_WEBHOOK_URL}'`

### Task-specific configuration

For the `uploadData` and `syncData` tasks, you must have instance(s) of [IBM Cloud Object Storage](https://www.ibm.com/cloud/object-storage) provisioned and the relevant buckets created.

For the `runDataFlow` task, you must use [Data Refinery](https://dataplatform.cloud.ibm.com/docs/content/refinery/data_flows.html), included as part of the IBM Watson Knowledge Catalog and IBM Watson Studio products. The Data Flow must already be defined before invoking the task to run the Data Flow.

For the `deployCF` task, you must set up a [Cloud Foundry account](https://www.ibm.com/cloud/cloud-foundry) before you can deploy your application into it.

For the `postSlack` task, you must set up a Slack Incoming Webhook URL, as
described here: [https://api.slack.com/incoming-webhooks](https://api.slack.com/incoming-webhooks)

For the `postEmail` task, you must have an SMTP server configured.

For the `triggerJenkins` task, you must have a Jenkins server configured.

For the `triggerTravis` task, you must have a Travis CI server configured.

## Integration of IBM AI OpenScale Model Operation Task CLI with CI/CD Tools

You can integrate the IBM AI OpenScale model operation task CLI tools into existing CI/CD tools, such as IBM Cloud Continuous Delivery service, Jenkins, and Travis CI.

### IBM Cloud Continuous Delivery service
Following are the typical steps to integrate the IBM AI OpenScale model operation task CLI tools with the IBM Cloud Continuous Delivery service.

1.  Create an account on [https://github.com](https://github.com), or on your enterprise GitHub service if you don't have an account yet.

1.  Create a repository on the GitHub service. Download the IBM AI OpenScale model operation task CLI tools `*tar.gz` file. Untar the `*tar.gz`, and add all untarred files, including the `modelops.sh` and   `model-operation-tasks-cli.jar` files to the repository. Add additional files, such as customized task-execution scripts, model-training source codes and model-scoring payloads to the repository.

1.  Create an account on [IBM Cloud](https://console.bluemix.net) if you don't have one yet. Log into IBM Cloud with your account.

1.  From the **Catalog** tab, search for and select *Continuous Delivery*; create an instance of this service.

1.  Go to your [dashboard](https://console.bluemix.net/dashboard/apps), and find your newly created *Continuous Delivery* service instance. Click to open it.

1.  Find the `DevOps` link directly above the service name, and click on it to get to the *Toolchains* page.

1.  Click **Create a Toolchain**, then select the *Build your own toolchain* tile.

1.  Provide a name for your toolchain, and click **Create**.

1.  Now, select **Add a Tool**, find the *GitHub* tile, and click on it.

1.  Fill in the GitHub Server field with your GitHub URL, such as [https://github.com](https://github.com), set the Repository type to *Existing*, and fill in the Repository URL with your repository name, such as `https://github.com/your_account/ibm-cloud-cd-demo.git`.

1.  Click **Create Integration**. You have added the Github tool to the toolchain.

1.  From the toolchain you created, click **Add Tool** again, find the *Delivery Pipeline* tile, and click on it. Then click **Create Integration**.

1.  Go back to the toolchain you just created. Select it and you will see the *GitHub* and *Delivery Pipeline* tools are configured for you.

1.  Click on Delivery Pipeline. Select the **Add Stage** button and provide a name. Select `Git repository` as the *Input* type, then fill in your repository information under *Git URL*. Select `master` for the *Branch*.

1.  Select the **JOBS** tab, select **ADD JOB**, then choose `Build`. Select the `Shell Script` option for *Builder type*.

1.  Now, in the *Build script* section, you can add your model operation CLI tool invocation scripts. For example, to invoke a script such as  `deploy_score_notify.sh` that will deploy a model, score the deployed model, and send a Slack notification, you can add the following line to the *Build script* section (assuming the scripts `install_bx_ml.sh` and `deploy_score_notify.sh` have been pushed to your GitHub repository):

    ```
    ./install_bx_ml.sh
    ./deploy_score_notify.sh ${MODELID}

    The script install_bx_ml.sh is needed to set up bluemix CLI tool and machine-learning plugins on the CI/CD environment.

    The ${...} are the environment variables tailored to your environment.  You can predefine them in the STAGE->ENVIRONMENT PROPERTIES section from the Delivery Pipeline tool.
    ```
    The script `install_bx_ml.sh` can have the following contents:

    ```bash
    #!/bin/bash
    hasBX=`which bx`
    if [ "${hasBX}"x = "x" ]
    then
       echo "Installing bx"
       curl -fsSL https://clis.ng.bluemix.net/install/linux | sh
    fi

    hasML=`bx plugin list | grep machine-learning`
    if [ "${hasML}"x = "x" ]
    then
       echo "Installing ML"
       bx plugin install machine-learning
    fi
    ```
    The script file `deploy_score_notify.sh` can have the following contents as an example:

    ```bash
    #!/bin/bash

    export ML_ENV=https://us-south.ml.cloud.ibm.com
    export ML_USERNAME=[Your WML instance username]
    export ML_PASSWORD=[Your WML instance password]
    export ML_INSTANCE=[Your WML instance ID]
    export SLACK_INCOMING_WEBHOOK_URL='https://hooks.slack.com/services/[Your slack webhook token]'
    export PATH=$PATH:.

    bx ml deploy $1 'Description' > deployResult.txt 2>&1
    DEPLOYMENT_GUID=`cat deployResult.txt | grep DeploymentId | awk '{print $2}'`
    echo "DEPLOYMENT_GUID=$DEPLOYMENT_GUID"
    status=`cat deployResult.txt | grep Status | awk '{print $2}'`
    echo "Deploy_Status=$status"
    if [[ "$status" == "DEPLOY_FAILURE" ]]
    then
    ./modelops.sh postSlack --channel '[Your slack channel]' --text 'model deployment failed' --url ${SLACK_INCOMING_WEBHOOK_URL}
    exit -1
    fi

    cp scoring_data_tensor_flow_ml.json scoring_data_tensor_flow_ml_new.json
    sed -i "s/MODEL_ID/$1/g" scoring_data_tensor_flow_ml_new.json
    sed -i "s/DEPLOYMENT_ID/${DEPLOYMENT_GUID}/" scoring_data_tensor_flow_ml_new.json
    bx ml score scoring_data_tensor_flow_ml_new.json > scoreResult.txt 2>&1
    scoreStatus=`cat scoreResult.txt | grep -c "[Your expected result]"`
    if [[ $scoreStatus -eq 0 ]]
    then
      ./modelops.sh postSlack --channel '[Your slack channel]' --text 'model scoring failed' --url ${SLACK_INCOMING_WEBHOOK_URL}
      exit -1
    fi

    ./modelops.sh postSlack --channel '[Your slack channel]' --text 'model deployment and scoring completed, with the expected prediction' --url ${SLACK_INCOMING_WEBHOOK_URL}
    ```

1.  Save your changes.

The integration is complete. You can click the **Run** icon to start the stage, which will execute your shell script that invokes the model operation task CLI command. Whenever there is a push to the Git repository, the Delivery Pipeline build will also be triggered, which in turn invokes the model operation task CLI command.

### Jenkins

Following are the typical steps to integrate the IBM AI OpenScale model operation task CLI tools with a Jenkins automation server.

1.  Create an account on [https://github.com](https://github.com) or on your enterprise GitHub service if you don't have an account yet.

1.  Create a repository on the GitHub service. Download the IBM AI OpenScale model operation task CLI tools `*tar.gz` file. Untar the `*tar.gz`, and add all untarred files, including the `modelops.sh` and   `model-operation-tasks-cli.jar` files to the repository. Add additional files, such as customized task-execution scripts, model-training source codes and model-scoring payloads to the repository.

1.  Set up your Jenkins server or gain access to an existing Jenkins server where you want to integrate with IBM AI OpenScale model operation task CLI tools.

1.  Log on to your Jenkins server.

1.  Click **Jenkins** find the *New Item* link, and click on it.

1.  Enter an item name, select *Freestyle project*, and click **OK**. Your Jenkins project is created.

1.  Click the **Source Code Management** tab, select *Git*, and fill in the Repository URL with your git repository where you host the CLI tool; provide your credentials in the Git repository.

1.  Click the **Build Trigger** tab. You can select *Trigger builds remotely*, which allows you to use Jenkins' REST API to invoke the Jenkins project. You will also need to provide the Jenkins API token associated with your account.

1.  Click the **Build** tab, click the *ADD BUILD STEP* drop-down list, and select `Execute shell`.

1.  Under the *Execute shell*, you can add your model operation CLI tool invocation scripts. For example, to invoke a script such as `deploy_score_notify.sh` that will deploy a model, score the deployed model, and send a Slack notification, you can add the following line to the *Build script* section (assuming the `scripts install_bx_ml.sh` and `deploy_score_notify.sh` have been pushed to your GitHub repository):

    ```
    ./install_bx_ml.sh
    ./deploy_score_notify.sh ${MODELID}

    The script install_bx_ml.sh is needed to set up bluemix CLI tool and machine-learning plugins on the CI/CD environment.

    The ${...} are the environment variables tailored to your environment.  You can pre-define them in the STAGE->ENVIRONMENT PROPERTIES section from the Delivery Pipeline tool.
    ```

    **Note**: The contents of the `install_bx_ml.sh` and `deploy_score_notify.sh` scripts can be found in the previous section of this document.

1.  Save your changes.

The integration is complete. You can click the **Build Now** icon to start the Jenkins project which will execute your shell script that invokes the model operation tasks CLI command. Alternatively, you can trigger the Jenkins project by using the IBM AI OpenScale Model Operation Task CLI tool with the `triggerJenkins` task. You can also trigger the Jenkins project via Jenkins' REST API call, which in turn invokes the model operation tasks CLI command.

### Travis CI

Following are the typical steps to integrate the IBM AI OpenScale model operation CLI tools with Travis CI service.

1.  Create an account on [https://github.com](https://github.com) or on your enterprise GitHub service if you don't have an account yet.

1.  Create a repository on the GitHub service. Download the IBM AI OpenScale model operation task CLI tools `*tar.gz` file. Untar the `*tar.gz`, and add all untarred files, including the `modelops.sh` and   `model-operation-tasks-cli.jar` files to the repository. Add additional files, such as customized task-execution scripts, model-training source codes and model-scoring payloads to the repository.

1.  Add a `.travis.yml` file to the root path of your Git repository. The content should be tailored to your task. For example, to invoke a script such as `deploy_score_notify.sh` that will deploy a model, score the deployed model, and send a Slack notification, you can add the following line to the `.travis.yml` file (assuming the scripts `install_bx_ml.sh` and `deploy_score_notify.sh` have been pushed to your GitHub repository):

    ```java
    language: java
    sudo: required
    script:
    - cd cli-tool
    - ./install_bx_ml.sh
    - ./deploy_score_notify.sh ${MODELID}

    The script `install_bx_ml.sh` is needed to set up bluemix CLI tool and machine-learning plugins on the CI/CD environment.

    The ${...} are the environment variables tailored to your environment.  You can pre-define them in Travis CI as needed.
    ```

    The contents of the install_bx_ml.sh and deploy_score_notify.sh scripts can be found in the previous section.

1.  Create an account on [https://travis-ci.org](https://travis-ci.org) where you want to integrate with the IBM AI OpenScale model operation task CLI tools.

1.  Sign in with your account.

1.  Click your account -> Profile.

1.  From the Profile page, click the **Repositories** tab and find your repository you want to integrate with Travis CI; click to enable it to be accessible by Travis CI.

1.  Click the repository again; you will see a **Restart build** button. Click it to restart a build.

1.  The Travis CI build will be automatically started once there is a push to the Git repository. The build will execute the steps defined in the `.travis.yml`, which invokes the model deployment task in this example. Alternately, you can trigger the Travis CI build by using the IBM AI OpenScale Model Operation Task CLI tool with the `triggerTravis` task. You can also trigger the Travis CI build via a Travis CI REST API call.

## Troubleshooting

* The use of environment variable place holders does not work

  Ensure that you are exporting the environment variable and executing the `modelops.sh` within the same shell. If you do not export the environment variable correctly, it will resolve to a blank value. Also, ensure that you check that the spelling of the environment variable matches that of the place holder you specify.

  For example, suppose you are invoking a task to post a message to Slack and you are using an environment variable for the Slack Incoming Webhook URL. If the environment variable you use to hold this URL is named `MODELOPS_SLACK_INCOMING_WEBHOOK_URL`, be sure that you export its value before invoking the task, with the correct spelling:

  `export MODELOPS_SLACK_INCOMING_WEBHOOK_URL='https://somewebhook.slack.com'`
  `./modelops.sh postSlack --channel '#modelops-alerts' --text 'Finished training model: Test Model 1' --url '${MODELOPS_SLACK_INCOMING_WEBHOOK_URL}'`

* Invoking a task using a task properties file does not work

  When invoking a task using an input properties file, instead of supplying command-line arguments, you need to include the `--file` option. If you forget to include the `--file` option, the tool will misinterpret the argument as a `task-id`, rather than a `.properties` file name.

  `./modelops.sh --file post_slack.properties`

## FAQ

* Q. How does the IBM AI OpenScale CLI model operations tool relate to the IBM Cloud CLI tool and its machine-learning plugin?

  A. The IBM AI OpenScale CLI model operations tool allows you to invoke tasks that are complementary to the tasks that you can invoke from the IBM Cloud CLI tool and its machine-learning plugin. Whereas, the IBM Cloud CLI tool allows you to perform model-specific tasks that include training, storing, deploying, and scoring machine learning models, the IBM AI OpenScale CLI model operations tool allows you to perform additional tasks that can help you manage other lifecycle aspects of these models, including uploading and synchronizing data in IBM Cloud Object Storage buckets, running Data Refinery Data Flows to prepare and cleanse data, sending notifications when model related tasks complete, and triggering CI/CD jobs in Jenkins or Travis CI as part of a delivery pipeline.

* Q. What is an example of a how these tools can be used together?

  A. One example could be a scenario where you have the source code for your
machine learning model source controlled in a code repository, such as Git.
Once any change to this code occurs, you may want to trigger a chain of tasks
as follows:

  Using the IBM AI OpenScale CLI model operations tool:
    * upload any new test or training data into IBM Cloud Object Storage (`uploadData` task)
    * perform cleansing on the test or training data (`runDataFlow` task)
    * copy the cleansed data into a training data bucket (`syncData` task)

  Then, using the IBM Cloud CLI tool and its machine-learning plugin:
    * perform training of the model
    * store the model in the Watson Machine Learning repository
    * deploy the model as a Web Service
    * score the model, verifying that it produces the expected results

  Then, using the IBM AI OpenScale CLI model operations tool:
    * deploy an updated application that uses this model to Cloud Foundry (`deployCF` task)
    * Finally, you can send emails (`postEmail` task) and Slack notifications (`postSlack` task) in any part of the task chain, to notify people of task success or failures.
