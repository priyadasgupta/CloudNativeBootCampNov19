# Deployment to Cloud environments and Continuous Delivery

- [Deployment to Cloud environments and Continuous Delivery](#deployment-to-cloud-environments-and-continuous-delivery)
  - [1. Introduction](#1-introduction)
  - [2. Prerequisites](#2-prerequisites)
  - [3. Deployment of Shipping Service to Open Shift](#3-deployment-of-shipping-service-to-open-shift)
  - [4. Deployment of Shipping Service to IBM Cloud with delivery pipelines](#4-deployment-of-shipping-service-to-ibm-cloud-with-delivery-pipelines)
  - [4. Summary](#4-summary)

## 1. Introduction

This lab guides you through process of cloud native applications deployments to cloud environments. We will deploy previously created shipping service to Red Hat Open Shift cloud and to IBM Kubernetes service.
The following Continuous Delivery tools will be used:

- OpenShift Source To Image Streams
- IBM Cloud Delivery pipelines

## 2. Prerequisites

1. Github account (on github.com NOT github.ibm.com)
2. Successfully completed Lab2. If you decide to skip Lab2 please fork the following location to get Lab2 solution: [https://github.com/TejasP/CloudNativeBootCampNov19](https://github.com/TejasP/CloudNativeBootCampNov19)
3. Personal IBM Cloud account for build pipelines


## 3. Deployment of Shipping Service to Open Shift

Open Shift Source to Image streams hook to Git repository and deploy application on each source update.

1. To ensure, that node application uses right version of node in the cloud, you need to provide additional configuration in `package.json` file. Make sure, that your `package.json` has the following contents (`scripts` and `engines` parts are important):

    ```json
    {
        "name": "shipping-service-XX",
        "version": "0.0.1",
        "main": "app.js",
        "scripts": {
            "test": "jest tests",
            "start": "node src/app.js"
        },
        "engines": {
            "node": "8.11.3",
            "npm": "5.6.x"
        },
        "dependencies": {
            "axios": "^0.18.0",
            "express": "^4.16.3"
        },
        "devDependencies": {
            "nock": "^9.4.2",
            "sinon": "^6.1.3"
        }
    }
    ```

    Please replace the `XX` with the unique number. Actual version numbers for the dependencies might be different, but usually higher.

2. Create a `.s2i/environment` file with environment variables needed to run shipping service

   ```properties
    PORT=8080
    MICROS_PRODUCTS_URL=product-service-java.eu-gb.mybluemix.net/products
   ```

3. Commit and push created file to github repository
4. Navigate to [https://learn.openshift.com/playgrounds/openshift311/](https://learn.openshift.com/playgrounds/openshift311/) and initiate 60-minute Open Shift environment by clicking `Start Scenario`.
5. Switch to dashboard tab and login using `developer` username and password
6. Click `Create project` button and create project named `lab-cnb`
7. Create new application by clicking `Node.js` in Catalog. Please selecte pure node.js without any extra databases
8. Click through Node.js wizard and provide the following values in `2.Configuration` step:
   - Project: lab-cnb
   - Version: 10-latest
   - Application-name: shipping-service
   - Git repository: your repository with shipping service or `https://github.com/TejasP/CloudNativeBootCampNov19`
9. Click `Continue to project overview` in Step 3. You will see `shipping-service` application view with build process running in the bottom-right corner
10. Wait until build process is completed and click application url which would look similar to `http://shipping-service-lab-cnb.2886795271-80-kitek03.environments.katacoda.com`. The service should reply with default `Cannot GET /` error.
11. Add request parameters to the service to make it respond with shipping data `http://shipping-service-lab-cnb.2886795271-80-kitek03.environments.katacoda.com/shipping?itemId=AAA&type=overnight`
12. Make more changes in GitHub repository and observe application re-deployment activity in Open Shift
13. Browse through Dashboard to see deployment details

## 4. Deploment of Shipping Service to IBM Cloud using IKS pipeline.
1. Since you were working in the directory `shipping-service` that is in Github, please make sure that all your changes are pushed to Github 

2. Create a new toolchain. Log in to IBM Cloud Console (if possible please use your **personal** IBM Cloud account ) and navigate to `Dev Ops` using top-left menu. Click on `Create toolchain` and select `Develop a Kubernetes app`. Then provide the following values:

    - Toolchain name: `shipping-service-js-XX-toolchain`
    - Region: `Dallas`
    - Select a resource group: `Default`
    - Toolchain name: `shipping-service-js-XX-toolchain`
    - Region: `Dallas`
    - Select a resource group: `Default`
    - Select a source provider as `GitHub`
    Under Tools Integration :
    - GitHub Server: github.com
    - Repository type: existing
    - Repository URL: [URL of shipping-service in your GitHub repository or just a https://github.com/TejasP/CloudNativeBootCampNov19]
    - Uncheck "Endable GitHub issues" and "Track developement of code changes"
    Delivery Pipeline:
    - App Name : shipping-service-js-XX
    - IBM Cloud API key : Your own IBM API key
    All remaining fields will be auto populated . validate it.
    
    
## 5. Deployment of Shipping Service to IBM Cloud with delivery pipelines

Delivery pipeline provides integration between code repository such as GitHub and IBM Cloud environment. Detailed deployment process:

1. Since you were working in the directory `shipping-service` that is in Github, please make sure that all your changes are pushed to Github 

2. Create a new toolchain. Log in to IBM Cloud Console (if possible please use your **personal** IBM Cloud account ) and navigate to `Dev Ops` using top-left menu. Click on `Create toolchain` and select `Build your own toolchain`. Then provide the following values:

    - Toolchain name: `shipping-service-js-XX-toolchain`
    - Region: `Dallas`
    - Select a resource group: `Default`
    - Select a source provider as `GitHub`
   
   Click `Create` button to create the toolchain
3. Create GitHub integration to the toolchain. Click `Add a tool` and select `GitHub`(not GitHub Enterprise Whitewater). Enter the following values for the new GitHub integration:

    - GitHub Server: github.com
    - Repository type: existing
    - Repository URL: [URL of shipping-service in your GitHub repository or just a https://github.com/TejasP/CloudNativeBootCampNov19]
  
    Uncheck "Enable GitHub Issues", check "Track deployment of code changes" and click "Create Integration"

4. Add `Delivery pipeline` tool to the toolchain:
    - Pipeline name: shipping-service-js-XX-pipeline
    - Show apps in the View app menu: yes

    You should now see two tiles in you toolchain view: GitHub and Delivery pipeline with both ticked as Configured
5. Create the pipeline by adding a stage to it. Click Delivery Pipeline tile and then `Add Stage` button. A new build `Stage configuration` window should appear to define pipeline's stage.
    - Name the stage `Build shipping-service `
    - For Input tab leave other fields as is
    - Switch to jobs tab and add a new job of type "Build":
        - Builder type: Container registry
        - Pipeline image version: Inherited from Configure Pipeline (latest)
        - API Key: `yll77n27jKMJ6S_6K-jPBmzMQ8SUi9TTjJVbd40zY9h-`
        - Cloud region: United Kingdom
        - Account name: `Cloud App Dev's Account`
        - Container registry namespace: `lab-cnb`
        - Docker image name: shipping-service-js-**XX**(replace XX with your suffix)
        - Build script: replace existing content with:
            ```shell
            #!/bin/bash
            echo -e "Build environment variables:"
            echo "REGISTRY_URL=${REGISTRY_URL}"
            echo "REGISTRY_NAMESPACE=${REGISTRY_NAMESPACE}"
            echo "IMAGE_NAME=${IMAGE_NAME}"
            echo "BUILD_NUMBER=${BUILD_NUMBER}"

            # Learn more about the available environment variables at:
            # https://cloud.ibm.com/docs/services/ContinuousDelivery?topic=ContinuousDelivery-deliverypipeline_environment#deliverypipeline_environment

            # To review or change build options use:
            # ibmcloud cr build --help

            echo -e "Checking for Dockerfile at the repository root"
            if [ -f Dockerfile ]; then 
            echo "Dockerfile found"
            else
                echo "Dockerfile not found"
                exit 1
            fi

            echo -e "Building container image"
            set -x
            ibmcloud cr build -t $REGISTRY_URL/$REGISTRY_NAMESPACE/$IMAGE_NAME .
            set +x
            ```

        - Working directory: leave empty
        - Build archive directory: leave empty
        - Enable test report: no
        - Enable code coverage report: no
        - Stop running this stage if this job fails: yes
    - Save the stage and run it. You can observe logs of the run and results docker image build execution by clicking on build details. When build is complete its artifacts are available for download

6. Update the pipeline by adding Deploy stage to it. Click Delivery Pipeline tile and then `Add Stage` button. A new build `Stage configuration` window should appear to define pipeline's stage.
    - Name the stage `Deploy shipping-service`, set the following values:
        - Input type: `Build artifacts`
        - Stage: `Build shipping-service`
        - Job: Build
        - Stage trigger: `Run jobs when the previous stage is completed`
    - Switch the Jobs tab and add a new job of type `Deploy` with the following values:
        - Deployer type: `Kubernetes`
        - Pipeline image version: `Inherited from Configure Pipeline (latest)`
        - API Key: `yll77n27jKMJ6S_6K-jPBmzMQ8SUi9TTjJVbd40zY9h-`
        - IBM Cloud Region: `United Kingdom`
        - Account name: `Cloud App Dev's Account`
        - Resource group: `Default`
        - Cluster name: `cnb-cluster`
        - Deploy script:
            ```shell
            #!/bin/bash
            #set -x

            ibmcloud ks cluster config --cluster cnb-iks

            export KUBECONFIG=~/.bluemix/plugins/container-service/clusters/cnb-iks/kube-config-lon02-cnb-iks.yml

            # Execute the file
            echo "KUBERNETES DEPLOYMENT COMMAND:"
            echo "kubectl apply -f deployment.yaml"
            kubectl apply -f deployment.yaml
            echo ""

            echo "DEPLOYED PODS:"
            kubectl get pods -n=lab-cnb
            echo ""

            # Execute the file
            echo "KUBERNETES SERVICE COMMAND:"
            echo "kubectl apply -f service.yaml"
            kubectl apply -f service.yaml
            echo ""

            echo ""
            echo "DEPLOYED SERVICES:"
            kubectl get services -n=lab-cnb
            echo ""


            # Execute the file
            echo "KUBERNETES INGRESS COMMAND:"
            echo "kubectl apply -f ingress.yaml"
            kubectl apply -f ingress.yaml
            echo ""

            echo ""
            echo "DEPLOYED ingress:"
            kubectl get ingress -n=lab-cnb
            echo ""
            ```

        Save the stage and run it. You can check deployment log output by clicking on stage's tile.
7. If deployment run successfully, the service should be now available at [http://shipping-service-XX.cnb-iks.eu-gb.containers.appdomain.cloud/shipping?itemId=RRR&type=standard](http://shipping-service-XX.cnb-iks.eu-gb.containers.appdomain.cloud/shipping?itemId=RRR&type=standard). Don't forget replacing `XX` with your application suffix
8. You can also try updating shipping-service code, push the changes to GitHub and observe that delivery pipeline picks the changes, builds a new version of service and deploys it to IBM Cloud

## 4. Summary

By now you should be able to deploy applications to IBM Cloud Kubernetes Service using both command line interface and Continuous Deployment services of IBM Cloud.

As seen at labs execution, deployment of javascript application is the same, as Docker image has all runtime needed to run the application
