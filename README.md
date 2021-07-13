# Deploying AppDynamics Java Agents with Cisco Intersight Service for Terraform on vSphere Infrastructure 
## Contents
        Use Cases
        Pre-requisites, Guidelines
        Intersight Target configuration for AppDynamics and on prem entities
        Retrieve Java agent artifacts using AppDynamics Controller ZFI API's leveraging TFCB
        Deploy a Java application stack and instrument with Java agent leveraging TFCB
        Try with a Sandbox


### Use Cases
* As a DevOps and App developer, use IST (Intersight Service for Terraform) to retrieve and install AppDynamics java agents together with the Java Apps
* As DevOps and App Developer, use Intersight and AppDynamics to get app and infrastructure insights for Full Stack Observability

![alt text](https://github.com/prathjan/images/blob/main/appduse.png?raw=true)

### Pre-requisites, Guidelines

1. The VM template that you provision in Step 7 below will have a user "cisco/Cisco123" provisioned

### Steps

1. Sign up for a user account on Intersight.com. You will need Premier license as well as IWO license to complete this use case. 

2. Sign up for a TFCB (Terraform for Cloud Business) at https://app.terraform.io/. Log in and generate the User API Key. You will need this when you create the TF Cloud Target in Intersight.

3. You will need access to a vSphere infrastructure with backend compute and storage provisioned

4. You will log into your Intersight account and create the following targets. Please refer to Intersight docs for details on how to create these Targets:

    Assist
    vSphere
    AppDynamics
    TFC Cloud
    TFC Cloud Agent - When you claim the TF Cloud Agent, please make sure you have the following added to your Managed Hosts. This is in addition to other local subnets you may have that hosts your kubernetes cluster like the IPPool that you may configure for your k8s addressing:
    NO_PROXY URL's listed:

            github-releases.githubusercontent.com
            github.com
            app.terraform.io
            registry.terraform.io
            releases.hashicorp.com
            archivist.terraform.io


5. If you are leveraging CiscoDevNet organization in app.terraform,io, please go to Step 7. Else go to Step 6.

6. If you have your own TFCB organization and would like to use that, you will have to change the terraform configuration to account for this. Please clone the following repos and create your own corresponding GIT repos. Look for CiscoDeNet org references in the TF files and replace with your own organization:

        Clone following repos:

        https://github.com/CiscoDevNet/appd_globalvar.git

        https://github.com/CiscoDevNet/appd_appstack.git


        With CiscoDevNet TFCB org:

        data "terraform_remote_state" "global" {

            backend = "remote"

            config = {

                organization = "CiscoDevNet"

                workspaces = {

                name = var.globalwsname

                }

            }

        }

        With your own TFCB org:

        data "terraform_remote_state" "global" {

            backend = "remote"

            config = {

                organization = "Lab14"

                workspaces = {

                name = var.globalwsname

                }

            }

        }

You will set up the following workspaces in TFCB and link to your newly created VCS repos. You will set the execution mode as noted below. Also, please use the workspace names provided since there may be dependencies defined around it:

    appd_globalvar -> <your_repo_appd_globalvar.git> -> Execution mode as Remote

    appd_appstack -> <your_repo_appd_appstack.git> -> Execution mode as Agent

Go to Step 8.

7. You will set up the following workspaces in TFCB and link to the VCS repos specified. You will set the execution mode as noted below. Also, please use the workspace names provided since there are dependencies defined around it:

    appd_globalvar -> <your_repo_appd_globalvar.git> -> Execution mode as Remote

    appd_appstack -> <your_repo_appd_appstack.git> -> Execution mode as Agent


8. You will open the workspace "appd_globalvar" in TFCB  and add the following variables specific to your AppDynamics environment. This workspace connects to the AppDynamics Controller specified her and downloads the commmands to instrument the Java applications:

    appname = "Supercar-Trader". This is the application name in your AppDynamics Console.

    accesskey = "XXXXXX".  This is your AppDynamics Access Key for API client access.

    zerover="21.6.0.232". This is the Zero agent version that you would like to instrument with.

    infraver="21.5.0.1784". This is the Infra agent version that you would like to instrument with.

    machinever="21.6.0.3155". This is the Machine agent version that you would like to instrument with.

    ibmver="21.6.0.32801". This is the IBM Java agent version that you would like to instrument with.

    javaver="21.5.0.32605". This is the Sun Java agent version that you would like to instrument with.

    url="https://devnet.saas.appdynamics.com". This is the URL of your AppDynamics Controller.

    clientid="YYYYYY". This is the API client ID to access your AppDynamics Controller.

    clientsecret="ZZZZZZ". This is the API client secret to access your AppDynamics Controller.

    A sample configuration would be as follows:

    ![alt text](https://github.com/prathjan/images/blob/main/appdapi.png?raw=true)

    Please also set this workspace to share its data with other workspaces in the organization by enabling Settings->General Settings->Share State Globally.


9. You will execute the workspace "appd_globalvar" in TFCB.  A successful Run should show the output as follows:

![alt text](https://github.com/prathjan/images/blob/main/appdapirun.png?raw=true)

You now have the AppDynamics ZFI API outputs for the download and install commands for the Java agent instrumentation to be used in subsequent steps.

7. You will open the workspace "appd_appstack" and add the following variables:

vsphere_server = "10.88.168.24". This is the vSphere server IP.

datacenter = "Piso14-Lab". This is your on prem datacenter which will host your application.

resource_pool = "ccmsuite". This is the resource pool to be used for VM provisioning.

datastore_name = "CCPHXM4". This is the data store to be used for VM provisioning.

network_name = "vm-network-6". This is the network interface to be used for VM provisining.

template_name = "ubuntu-tmp". This is the template to be used for VM provisioning.

vm_folder = "terraform". This is the folder in vSphere for VM hosting.

vm_prefix = "terraform-". This is the prefix to be used for VM provisioning.

vm_domain = "lab14.lc". This is the vSphere domanin for VM provisioning.

vm_cpu = 2.  This is the CPU profile for the VM.

vm_count = 1. This is the number of VM's to be provisioned.

root_password = "XXXX". This is the root password for vSphere access.

vsphere_user = "administrator@vsphere.local". This is the user name for vSphere access.

vsphere_password = "YYYYY". This is the user passsword for vSphere access.

vm_memory = 4096   . This is the memory for the VM.

A sample configuration would be as follows:

![alt text](https://github.com/prathjan/images/blob/main/appdvm.png?raw=true)

8. You will verify that this workspace uses the Terraform Cloud Agent provisioned on your on prem infrastructure. You will execute the workspace "appd_appstack" in TFCB. A successful Run should show the output as follows. Executing this workspace installs a sample Java AppStack based on Tomcat with the AppDynamics Java agent instrumentation. The instrumentation of the AppDynamics ZeroAgent will automatically instrument any Java application that is deployed on Tomcat as well without the need for any manual instrumentation. We will take a look at a sample Java application deployment and auto instrumentation in Step 10.

![alt text](https://github.com/prathjan/images/blob/main/appdvmrun.png?raw=true)



9. Make a note of the VM IP in the previous step and open the Tomcat Manager Console for a sample Java Application deployment which will automatically instrument it with the AppDynamics Java agent:

![alt text](https://github.com/prathjan/images/blob/main/tomcat.png?raw=true)

10. You will now deploy a sample Java application on Tomcat and restart Tomcat so AppDynamic Zero Agent can instrument the app with the Java agent. 
Use: 

     /Supercar-Trader
     file://opt/appdynamics/DevNet-Labs/applications/Supercar-Trader/Supercar-Trader.war

![alt text](https://github.com/prathjan/images/blob/main/tomapp.png?raw=true)

11. To see the application insights in AppDynamics Console, you should initiate some load on the deployed services so the Java agent installed can send the metrics back to the AppDynamics Controller:

ssh cisco@<vm-ip>

Cisco123

Execute: /opt/appdynamics/DevNet-Labs/applications/Load-Generator/phantomjs/start_load.sh

12. You will now log into AppDynamics console and examine the insights for the Java Application that you just deployed: 

![alt text](https://github.com/prathjan/images/blob/main/appdapp.png?raw=true)

### De-provisioning

You can decommision all resources provisioned by queing a destroy plan in each workspace. 

Destroy and remove from TFCB: appd_globalvar

Destroy and remove from TFCB: appd_appstack

### Try with a Sandbox

A sandbox covering a lot of the above concepts can be found here:

TBD