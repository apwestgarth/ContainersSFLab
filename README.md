# ContainersSFLab Repo

This repository contains all the needed files and instructions to go through an ASP.NET lift and shift with Windows containers and Service Fabric lab.

Requirements for running the lab are

- Windows 10 Enterpsire or Professional Edition or Windows Server 2016
- Docker for Windows
- Visual Studio 2017 15.7
- Service Fabric SDK 3.1 and Service Fabric Runtime 6.2
- An Azure subscription:
    - If you do not have one, you can get a trial subscription here: https://azure.microsoft.com/en-us/free/

# Containerize an existing ASP.NET application using Visual Studio and Service Fabric
In order to lift and shift an application to the cloud without re-writing code, we're going to containerize the application and deploy it to a Service Fabric cluster.

# Lab Overview

In this lab, you will be guided through the following tasks:

1. Create a Service Fabric application by containerizing an existing ASP.NET web
    application though Visual Studio

1. Deploy your application to a Service Fabric cluster

1. Upgrade and optimize the way your application runs in the cluster

*This lab's instructions can also be accessed at https://aka.ms/sfcontainerlab along with necessary resources.*
  
# Create a Service Fabric application with your container

**Goal:** The goal of this section is to have the container running as a service in a Service Fabric application.


## Process


| **Step**                                         | **Procedure**                                                                                                                                |
| ------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Run the application                              | Run the existing ASP.NET application to verify its functionality, using Visual Studio and IIS Express.                                       |
| Create a Service Fabric Application              | Use Visual Studio tooling to create a Service Fabric application hosting the container.                                                      |
| Run the container as a service in Service Fabric | We will set up a local development cluster and deploy the containerized application to the local cluster.                                    |
| Apply container policies and upgrade             | We will apply container resource policies and parameters for configuration. Finally, we will upgrade the running application in the cluster. |

## Run the Application

In this section we will test the ASP.NET application using Visual Studio and IIS Express.

| **Step** | **Action** | **Result** |
| -------- | ---------- | ---------- |
| 1 | Open the solution **eShopLegacyWebForms.sln**, which you will find in the **\\Lab\\eShopLegacyWebFormsSolution** folder on the desktop. | Visual Studio will open, and you will see one project with an ASP.NET MVC application in Visual Studio.|
| 2 | Change the **Solution Configuration** to **Debug** (dropdown in the menu at the top) and press **F5** (Or the green *IIS Express* button) to start debugging the application. | The application will now build, and Edge will open with the application when it’s ready. | 
| 3 | The application simulates a simple web shop administrative interface. Feel free to browse around the application to see details of the various products and some of the functionality. | **Note:** The application is setup to use mock data loaded from source files, so all changes will be reverted once the debug session closes. |
| 4 | Back in VS, press **Shift+F5** or the red stop button at the top of your VS window to stop the debugging | Debugging stops. |

### Completion

You’ve now completed testing an ASP.NET application using Visual Studio and IIS Express.

## Create a Service Fabric Application

In this section we will create a Service Fabric application with the container as a service, and configure it.

<table>
<thead>
<tr class="header">
<th><strong>Step</strong></th>
<th><strong>Action</strong></th>
<th><strong>Result</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td>In the Solution Explorer on the right of the VS window, <strong>Right-Click</strong> the project file (eShopLegacyWebForms) and choose <strong>Add -&gt; Container Orchestrator Support</strong>. Choose <strong>Service Fabric</strong> in the dropdown menu.</td>
<td>Visual Studio 2017 will now create the required Dockerfile for your container image, as well as a Service Fabric application project in the solution in Visual Studio. Check this by confirming that there is a *eShopLegacyWebFormsApplication* Service Fabric application in your Solution Explorer.</td>
</tr>
<tr class="even">
<td>2</td>
<td>Find and open the <strong>Dockerfile</strong> in your project files. You'll see that it is using a windowservercore image for the underlying OS in the container by default - one that is compatible with being deployed on your developer machine.</td>
<td>VS creates a Dockerfile which you can modify as your requirements change, to change the container image being generated when you build your code</td>
</tr>
<tr class="odd">
<td>3</td>
<td><p>To instruct Service Fabric to open a specific port (4000) for the service (our container), we must specify the port to use in the ServiceManifest.xml file.</p>
<ol type="1">
<li><p>Open <strong>ServiceManifest.xml</strong>, which is in the PackageRoot folder of the eShopLegacyWebForms project</p></li>
<li><p>Specify the port to use for the endpoint in the &lt;Endpoint&gt; element (near the bottom) as 4000. Here is what that should look like: </p></li>

```csharp
Endpoint Protocol="http" Name="eShopLegacyWebFormsTypeEndpoint" Type="Input" Port="4000";
```

<li><p>Save the file</p></li>
</ol>
</td>
<td>The endpoint is bound to the service, which will host the container in Service Fabric. This configuration will ensure that Service Fabric attaches a port to the service process when it’s running. <br><strong>Note:</strong> If no endpoint is specified, Service Fabric will assign a random endpoint from a predefined pool of ports.</td>
</tr>
<tr class="even">
<td>4</td>
<td><p>To instruct Service Fabric to map port 80 of the container to the endpoint we specified above, we must create a PortBinding configuration in the ApplicationManifest.xml file.</p>
<ol type="1">
<li><p>In the ApplicationPackageRoot folder under the Application project (eShopLegacyWebFormsApplication1), open the ApplicationManifest.xml file</p></li>
<li><p>Verify the following configuration, as part of the &lt;ContainerHostPolicies&gt; element:</p></li>

```csharp
PortBinding ContainerPort"80" EndpointRef="eShopLegacyWebFormsTypeEndpoint"
```

<li><p>If not - add it to your manifest (under ServiceManifestImport > Policies > ContainerHostPolicies) and save the file</p></li>
</ol></td>
<td>This configuration ensures that the port exposed by the container will be mapped to the endpoint of the service.
</tr>
<tr class="odd">
<td>5</td>
<td><p>Finally, we’ll specify an environment variable to pass to the container.</p>
<ol type="1">
<li><p>In the ServiceManifest.xml file, add the following element in the CodePackage element to pass an environment variable to the container that defines the name of the shop (you can modify the "Value" if you'd like):</p></li>


```csharp
<EnvironmentVariables>
    <EnvironmentVariable Name="eShopTitle" Value="on SF!"/>
</EnvironmentVariables>
```

<li><p>Save the file</p></li>
</ol>
<p><strong>Note:</strong> If the environment variable value you put in is too long a string, it will cause the web site to not show any title – so be kind.</p></td>
<td>This environment variable configuration will be used for the title displayed on the front page of the web application.</td>
</tr>
</tbody>
</table>

### Completion

In this section of the lab, we’ve created a Service Fabric application that consists of a containerized ASP.NET + IIS application, and configured it to run in a Service Fabric cluster.

## Run the container in Service Fabric

In this section we will run the container in Service Fabric on our dev machine.
You can run a Service Fabric cluster on a Windows 10 with the Service Fabric SDK, which provides cluster configuration scripts that support setting up clusters of one node or five nodes for development purposes.

<table>
<thead>
<tr class="header">
<th><strong>Step</strong></th>
<th><strong>Action</strong></th>
<th><strong>Result</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td><p> Let’s start by creating a one node cluster. If you hover the little Service Fabric icon in your notification tray (bottom right of your OS UI) - you will see that it says "Service Fabric Local Cluster Manager (1 Node). <strong>Right-click</strong> on it and select <strong>Start Local Cluster</strong> to start the local cluster. <br> Click <strong>Yes</strong> if asked to grant admin permission to the process.</td>
<td><p>A Service Fabric cluster consistent of a single node will now be created on your developer machine. We use a single node in this lab as additional nodes will only add overhead to the developer experience are usually only needed for debugging failover scenarios.</p></td>
</tr>
<tr class="even">
<td>2</td>
<td>
<p>After a few minutes, the Local Cluster Manager will push an update that it has successfully connected to the new cluster. <strong>Right-click</strong> on the Local Cluster Manager icon in the tray and click on <strong>Manage Local Cluster</strong>. </p> 
<p>This will open up Service Fabric Explorer (SFX). Once in SFX, find the slider the top right that defines the refresh rate, and move it all the way to the right (2 sec)<p></td>
<td><p>SFX is your management UI for any Service Fabric cluster, locally, in Azure, or on-premises.</p>
<p>Service Fabric explorer let you get information and updates about your cluster, applications, and services, and provides an interface for doing administrative commands against the cluster. Speeding up the rate at which the UI helps you keep a better track of how your cluster is handling your application, which is especially helpful when developing and testing. Feel free to click around the UI to get a feel for what it shows. </p></td>
</tr>
<tr class="odd">
<td>3</td>
<td><p>Next go back to VS to publish the application:</p>
<ol type="1">
<li><p><strong>Right-click</strong> the Service Fabric application project and select <strong>Publish</strong> to publish the application to the local Service Fabric cluster.</p></li>
<li><p>Choose <strong>PublishProfiles\Local.1Node.xml</strong> as the Target profile</p></li>
<li><p>Click <strong>Publish</strong></p></li>
<li><p>If VS prompts you to restart as Administrator, hit <strong>Yes</strong>, or if it wants to inform you that files have changed externally, uncheck the *reload all files* prompt and click <strong>Yes to all</strong></p></li>
</ol></td>
<td>Visual Studio will build the container and deploy the application to Service Fabric. Watch the *Output* console in VS to see some of the steps being taken.</td>
</tr>
<tr class="even">
<td>4</td>
<td>Once the deployment is completed, go back to <strong>SFX</strong>, which will now show you one application deployed in the cluster. On the left nav, click all the way down into the application to explore the heirarchy of a SF application. Click on <strong>Node_0</strong> at the very bottom of the tree. This will pull up the running instance of the app on this node (which is your dev machine). <strong>Copy</strong> the URL that you see associated with your endpoint, and open it up in a new tab in your browser.

```
http://LAB01:4000
```

</td>
<td>You should see your eShop UI come up! In this scenario we are just publishing the container, however you can also debug the application code in a container running on Service Fabric, simply by pressing F5 in VS. The difference here is that we can go back to editing code now without shutting off the running instance and can then upgrade the application.</td>
</tr>
</tbody>
</table>

### Completion

In this section of the lab, we’ve created a local Service Fabric cluster
and deployed our application in a container to the cluster.

## Apply container policies and upgrade the application

In this section we will enable parameterization of our application
configuration and apply resource policies to the container. We will also roll-out an upgrade to the Service Fabric application.

Head back to VS!

<table>
<thead>
<tr class="header">
<th><strong>Step</strong></th>
<th><strong>Action</strong></th>
<th><strong>Result</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td><p>Docker and Service Fabric can control the amount of resources containers can use in a cluster. To specify the amount of memory and CPU for our container do the following:</p>
<ol type="1">
<li><p>In the <strong>ApplicationPackageRoot</strong> folder under the Application project, open the <strong>ApplicationManifest.xml</strong> file</p></li>
<li><p>Insert the following configuration, as part of the <strong>&lt;Policies&gt;</strong> element (under ContainerHostPolicies):</p></li>

```csharp
<ServicePackageResourceGovernancePolicy CpuCores="1" MemoryInMB="1024" />
```

<li><p>Save the file</p></li>
</ol></td>
<td>This will make sure this container gets a single CPU core and 1GB of memory assigned in the cluster. Service Fabric uses this enforcement to place containers across the cluster as your deployment scales up.</td>
</tr>
<tr class="even">
<td>2</td>
<td><p>We also want to have a single configuration file for changing configuration for our container across environments. In Service Fabric we will use the ApplicationParameter files to do this. To parameterize the environment variable, do the following:</p>
<ol type="1">
<li><p>Open the <strong>ApplicationManifest.xml</strong> file</p></li>
<li><p>Insert the following to override the environment variable in the ServiceManifest as part of the <strong>&lt;ServiceManifestImport&gt;</strong> element (just under the &lt;/Policies&gt; tag):</p></li>

```csharp
<EnvironmentOverrides CodePackageRef="Code">
    <EnvironmentVariable Name="eShopTitle" Value="[eShopLegacyWebForms_eShopTitle]" />
</EnvironmentOverrides>
```

<li><p>The value specified in square bracket `[]` is a reference to a parameter, which we will also have to add to the ApplicationManifest, as part of the <strong>&lt;Parameters&gt;</strong> element at the top of the file. We will set the default value to be an empty string.</p></li>

```csharp
<Parameter Name="eShopLegacyWebForms_eShopTitle" DefaultValue="" />
```
<li><p>Save the file</p></li>
</ol></td>
<td>Now you have a parameter set up to modify the title of the website. You can similarly add additional environment variables to provide configurations to help your services run the way you need them to, without constantly modifying your container images.</td>
</tr>
<tr class="odd">
<td>3</td>
<td><p>Now that we have defined the environment variable to be overridden by a parameter, let’s specify the value for the parameter:</p>
<ol type="1">
<li><p>In the <strong>ApplicationParameters</strong> folder, open the  <strong>Local.1Node.xml</strong> file. This is the file which will be used when we publish to our local 1 node cluster.</p></li>
<li><p>Add the following to the <strong>&lt;Parameters&gt;</strong> element:</p></li>

```csharp
<Parameter Name="eShopLegacyWebForms_eShopTitle" Value="MSReady" />
```

<li><p>Save the file</p></li>
</ol></td>
<td>Parameter files is a concept understood by Visual Studio and VSTS. If you will be using PowerShell or other means to deploy, Service Fabric accepts parameters as a hash table.</td>
</tr>
<tr class="even">
<td>4</td>
<td><p>Let’s go ahead and upgrade the application in the local cluster:</p>
<ol type="1">
<li><p><strong>Right-click</strong> the Service Fabric application project and select <strong>Publish</strong> to publish the application to the local Service Fabric cluster.</p></li>
<li><p>Choose <strong>PublishProfiles\Local.1Node.xml</strong> as the Target profile.</p></li>
<li><p>Check <strong>Upgrade the application</strong></p></li>
<li><p>Click <strong>Manifest Versions...</strong></p></li>
<li><p><strong>Expand the tree</strong> under eShopLegacyWebFormsPkg</p></li>
<li><p>Change the <stron> New Version</strong> of the <strong>Config</strong> to <strong>2.0.0</strong> since we have modified the configuration, but not the code</p></li>
<li><p>Click <strong>Save</strong></p></li>
<li><p>Click <strong>Publish</strong></p></li>
</ol></td>
<td>All parts of a Service Fabric application are versioned. This enables the platform to only upgrade the specific parts of the application which are being changed as part of an upgrade.</td>
</tr>
<tr class="odd">
<td>5</td>
<td><p>Open <strong>SFX</strong> and check out the following:
<ol type="1">
<li><p>Click <strong>Applications</strong> on the left nav and click on <strong>Upgrades in Progress</strong> on the top nav to see your new version coming through</p></li>
<li><p>When completed, head back to <strong>All applications</strong> and confirm that the application is now in version 2.0.0</p></li>
<li><p>Click <strong>Cluster</strong> and head to the <strong>Metrics</strong> page. Uncheck the <strong>Normalize metric data</strong> if it is checked, and select <strong>servicefabric:/_CpuCores</strong> in the metric options on the left of the chart. This should show the deployment at 1 core<p></li>
<li><p>Similarly, check on the memory metric data as well, which should show the deployment at 1024MB, as we specified<p></li>
</ol></td>
<td>If the cluster had consisted of multiple nodes, the upgrade would have taken significantly longer - each node gets updated to completion and then SF runs a series of health checks to make sure the app is stable before moving forward. If an error occurred, the upgrade would automatically roll-back, leaving the application up and running. </p>
<p> Checking out the Metrics on your cluster will also show you that your application is now consuming exactly one core in the node.</p></td>
</tr>
<tr class="even">
<td>6</td>
<td>Head back over to <strong>http://LAB01:4000</strong> to see the change reflected in the title.</td>
<td>With that you can see that your parameter successfully overrode the environment variable we set in the Service Manifest.</td>
</tr>
</tbody>
</table>

### Completion

In this section of the lab, we’ve parameterized our configuration,
applied resource governance and upgraded our application in the Service
Fabric cluster.

## Bonus steps!

Your eShop is getting quite popular! Let's scale this thing!

<table>
<thead>
<tr class="header">
<th><strong>Step</strong></th>
<th><strong>Action</strong></th>
<th><strong>Result</strong></th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td>1</td>
<td> <strong>Right-click</strong> on your Local Cluster Manager, and click on <strong>Switch Cluster Mode</strong>. Select <strong>5 node</strong>, and click <strong>Yes</strong> if prompted to confirm that you want to switch your cluster (of course you are sure!). This will take a couple of minutes.</td>
<td>This will shut down your current local cluster and start up an emulation of a 5 node cluster on your dev machine. This is done by 5x-ing all the processes required by SF to run on different nodes like FabricHost.exe and Fabric.exe. You will see SFX start to throw a bunch of failures at this point as well, since none of the APIs are returning values as the cluster is down. You can hover on the Local Cluster Manager icon in the tray to check the status at any point.</td>
</tr>
<tr class="even">
<td>2</td>
<td>When the cluster is back up, head over to SFX to confirm that you have 5 nodes running.</td>
<td>This info is available in the main dashboard or by expanding the Nodes tree on the left to show each running node in the cluster. You will also notice that your app is no longer running, since the entire cluster was removed.</td>
</tr>
<tr class="odd">
<td>3</td>
<td>Head back to VS to make the following changes to the application's 5 node publish profile. Open the <strong>Local.5Node.xml</strong> file, in the <strong>ApplicationParameters</strong> folder, and make the following changes:
<ol type="1">
<li><p>Add the following to the <strong>&lt;Parameters&gt;</strong> element:</p></li>

```csharp
<Parameter Name="eShopLegacyWebForms_eShopTitle" Value="MSReady" />
```

<li><p>In the same section, increase the value of the <strong>eShopLegacyWebForms_InstanceCount</strong> parameter from 1 to <strong>3</strong></p></li>
<li><p>Save the file</p></li>
</td>
<td>This adds back the environment variable override to this deployment configuration profile, since the previous modifications we did would only work for a 1 node cluster, and increases the number of instances to be deployed to 3.</td>
</tr>
</tr>
<tr class="even">
<td>4</td>
<td>Open up the <strong>ServiceManifest.xml</strong>, under the project's PackageRoot folder, and remove the "Port="4000" that we added in earlier from the Endpoint configuration, so that Service Fabric can now dynamically assign ports to each instance. It should now look like: 

```csharp
Endpoint Protocol="http" Name="eShopLegacyWebFormsTypeEndpoint" Type="Input";
```

</td>
<td>This is a requirement when deploying multiple instances of a service that listen on a port so as not to create a port clash with the 3 containers trying to sit on the same port address. By letting SF dynamically assign ports to each container, the 3 containers will all come up successfully and listen on different ports that you can check.</td>
</tr>
<tr class="odd">
<td>5</td>
<td>Publish the application to the cluster as before, but this time, set the Target Profile to <strong>PublishProfiles\Local.5Node.xml</strong>.</td>
<td>Your application will be built, and 3 containers will be deployed on your local cluster.</td>
</tr>
<tr class="even">
<td>6</td>
<td>Open up SFX and confirm that you have 3 instances of the container running by expanding out the app's tree as before, and checking what nodes the replicas are running on. Pick one of the nodes, and click on it to check the associated endpoint, and open up that endpoint in a new browser tab. It may look something like: http://LAB01:32001.</td>
<td>The 3 containers have come up and are running on 3 different endpoints. Each should bring up the eShop UI successfully</td>
</tr>
<tr class="odd">
<td>7</td>
<td>Your eShop is still getting more traffic and needs to handle more users! Let's scale up the number of instances some more by increasing the instance count of the service to "-1". Unlike with math, -1 in SF represents a "global service" in your cluster, or a service that needs to run on every node. 
<ol type="1">
<li>Click into your service's page on SFX in the app tree - look for <strong>fabric:/eShopLegacyWebFormsApplication/..</strong> under "fabric:/eShopLegacyWebFormsApplication" </li>
<li> Then, click on <strong>Actions</strong> (top right of the view) and select <strong>Scale Service</strong> </li>
<li>Set the new desired instance count to <strong>-1</strong>, and click <strong>Scale Service</strong></li>
</td>
<td>You will see a service update get kicked off, following which more nodes will start to show up under the list of nodes on which your container is running, on the fully expanded App tree on the left nav. That's it! You just 5x scaled your service in minutes! In SF, it is also possible ot set up autoscaling rules as well as programmatic scaling to automate this behavior - something that most customers opt for in volatile load handling production workloads. Feel free to test out any of the new containers by grabbing any of the new endpoints and opening them up in your browser as before.</td>
</tr>
</tbody>
<table>

### Completion

In this stage - you have succesfully (simulated) scaled your underlying infrastructure to 5 nodes, and then scaled the number of instances of your application deployed on your cluster both through the app config in VS as well as through SFX directly. 

# Next steps

From here, you have a few different things you should try to turn this into a production ready lift-and-shift.

* Push your container image to a container registry (Azure Container Registry or Docker Hub) instead of using a local registry as in this lab - this will enable you to control versioning as well as remove the dependency of constantly deploying from an environment where you have the full application code available (as with VS here). This also opens you up to now using "Docker compose" to deploy your container to a Service Fabric cluster!
* Add a new service to your application. Once you have moved your app to a container, you realize the need for a data analytics service on how your eShop is being used. In the same VS solution, you can add a Service Fabric service that will come with its own ServiceManifest.xml, which then has to imported into the same ApplicationManifest.xml we've been modifying - and you're all set.
* Deploy this to a cluster running in the cloud. Head over to aka.ms/tryservicefabric to get set up with a Service Fabric Party Cluster! These are free, time-limited clusters for you to party on.
* Experiment with Service Fabric Explorer - Standalone. This is SFX, fully open-sourced, running as a desktop application to you can connect to multiple clusters and customize the way that it runs. (If doing this lab on a LODS machine - SFX Standalone is pinned to your taskbar, the bigger SF logo present on your desktop.)

Thanks for spending the time to learn more about lifting and shifting workloads to Service Fabric! Feel free to reach out to me with any feedback on this or if you'd like to contribute to expanding this further!
