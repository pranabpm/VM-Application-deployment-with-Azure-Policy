# VM Application deployment with Azure Policy

Mission critical workloads often need to deploy hardened images to secure their application by pre-installing applications. This works most of the cases but increase over head of managing and maintaining the latest upgraded applications and reimaging. Azure VM Application a resource type under Azure compute Gallery that provides the flexibility to manage and deploy application across virtual machines and scalesets. It decouples application installation from VM Base image so you can streamline updates, reduce image maintanance overhead to accelerate deploiyment cycle. This functionality is best suited for deploying high scale, low latency, AI, micro-services, security and complient workloads.

This is great for managing application deployment on image but how to enforce all virtual machines or scalesets have a particular application installed or ensure all deployments in a subscription or respurce group have deployment of a particular application/s are of the most recent version. This can be achieved in conjunction of using Azure policy to enforce application deployment during VM deployment. In this post I will walk thrugh step by step process creating Comoute Gallery and Build a custom policy to enforce automated application installation during VM Provisioning. On a high level this PoC will cover:
      
      1) Creating VM Application in Compute Gallery.
      2) Apply VM Application during VM deployment.
      3) Automate VM application deployment using Azure Custom Policy.

**Creating VM Application in Compute Gallery**

1) Create Storage Account if you don’t have one and create a container or multiple containers for applications as needed. You can create one application for all application or create separate container for each application.

<img width="900" height="450" alt="Picture1" src="https://github.com/user-attachments/assets/a32d1b98-0a72-48d9-ac9f-3ab530cb1e1a" />

2) In the container will upload application installer file that we want to install on VM. In this PoC I am using Chrome msi application.
   
<img width="900" height="450" alt="Picture2" src="https://github.com/user-attachments/assets/7c4575dc-5ba8-4920-9c0e-d03c35777148" />

3) Now let’s create the application in Azure Compute Galleries. Unfortunately, we cannot upload files directly from local machine, so we have to upload the files to the storage container first before creating application in gallery. If you don’t have a compute gallery already create one. I am using the same region as the blob container.
 
 <img width="600" height="450" alt="Picture3" src="https://github.com/user-attachments/assets/c39b77cc-9a83-4e16-a7c3-df21e971e7cd" />
 
4) In the galleries ```Overview``` under ```+Add``` you will see two options:
   
      a.	```VM image definition```\
      b.	```VM application definition```
   
I will create VM application definition in this case and upload the application from the storage container. From ```Overview -> Add dropdown -> VM application definition```

<img width="900" height="490" alt="picture 17" src="https://github.com/user-attachments/assets/97331303-c46b-4060-b854-3aae6b0b1705" />

Complete the basic information and navigate to ```Publishing options```

<img width="600" height="450" alt="Picture4" src="https://github.com/user-attachments/assets/27f4daf9-54bc-48ae-9b39-a88e6b4d8738" />

5) Publishing definitions are optional field and can be filled as needed. For the PoC I am only adding a description. You may add tags as needed then ```Review + Create```

   <img width="800" height="450" alt="Picture5" src="https://github.com/user-attachments/assets/e1149b03-0a99-4db1-87f1-194c828a8060" />

7) Next, we will add an application version. The version will have the actual installable file.
   **Note:** An application definition can have multiple versions.

From galleries overview screen select the definition we just created then navigate to ```Add``` to fill in the details and browse the source application package we upload in the storage account. Add mandatory install and uninstall script and any additional optional field you like to add then hit ```Replication```. 

**Note:** Replication allows you to deploy this application to servers in different regions. You can add tag if you like then hit ```Review + create```.
  
  <img width="700" height="450" alt="Picture6" src="https://github.com/user-attachments/assets/1b6bb2b1-235b-4a6c-aa40-79ac4b4ee4ec" />

For this PoC I am going to use a single region.

<img width="900" height="450" alt="Picture7" src="https://github.com/user-attachments/assets/35d9725c-1671-46b1-8111-40975dd465d9" />

Post deployment the VM Application should be shown below with a version and published date.

<img width="700" height="450" alt="Picture8" src="https://github.com/user-attachments/assets/f0d20dd7-d96d-4a02-a7e1-965ae86ea392" />

**Apply VM Application during VM deployment**

7) Once the deployment is complete to test this out you can deploy a VM and see if the VM Application from the compute gallery is deployed successfully. To test this I am going to deploy a VM (Windows) in the same region as the replica. Follow along the prompts to select the default options for PoC all the way to ```Advanced``` tab.
   
<img width="900" height="450" alt="Picture9" src="https://github.com/user-attachments/assets/c2715ab1-cbbd-40d6-99e9-9af5d348b729" />

9) Select the application available to install in the region. Also notice that we have different versions we can select from. Select the application and click ```save```.
    
   <img width="700" height="450" alt="Picture10" src="https://github.com/user-attachments/assets/2dfd6a3d-87f3-4af8-8024-56175898ef91" />

11) If we have multiple applications selected to install, then we can select the order in which they will be installed. I am not selecting any preference for this PoC as I only have one application.
    
   <img width="900" height="450" alt="Picture11" src="https://github.com/user-attachments/assets/fddd4768-27cf-4129-99a0-71b212d6b103" />

13) Once the validation passes click ```Create```.
    
    <img width="700" height="450" alt="Picture12" src="https://github.com/user-attachments/assets/de48eeaf-ba28-413f-bfd7-319a95a28949" />

15) Login to your VM once deployed to verify the application is installed.
    
<img width="900" height="450" alt="Picture13" src="https://github.com/user-attachments/assets/7ae207bf-421b-4fca-b086-b838ceb87a5f" />


**Automate VM application deployment using Azure Custom Policy**

So far so good – Application installation during VM deployment worked. What if you have a scenario where you want to make sure any VM or scale set that will be deployed in your subscription/resource group should have a particular application/s installed. TL;DR – enforce a particular application are of the most recent version on each VM or scaleset provisioned.

This can be achieved using a DeployIfNotExist Azure policy. Let’s try it in the subsequent section.
For the PoC I am going to assign a policy at the Resource Group level to enforce a particular application (Chrome) with specific version deployed to each VM and scale set. The reference to the custom policy source code is available for reference and reuse. 

12) Create a policy under ```Policy -> Authoring -> Definition```. Add definition location, Name, and add a custom policy.

<img width="600" height="470" alt="Picture14" src="https://github.com/user-attachments/assets/7eb893cb-aa79-4326-8102-95b8be3104a6" />

13) Once the definition is successfully created assign to the scope you want this to apply. For the PoC I am assigning it to the resource group.

<img width="600" height="470" alt="Picture16" src="https://github.com/user-attachments/assets/022f2708-4717-4413-bfd8-781e26e42a80" />

14) Once assignment is done successfully validate by check the policy reflected under resource group.

<img width="800" height="450" alt="Picture15" src="https://github.com/user-attachments/assets/90f53c19-45a4-4ec1-a01e-db99c72cfa53" />

15) Validate the policy by creating a VM (Windows) and not selecting any VM application in advanced section. Policy will automatically enforce the application deployment if not exist and can take few minutes after VM deployment is successful.

<img width="900" height="450" alt="Picture13" src="https://github.com/user-attachments/assets/ad8bbb01-c908-4ca3-8256-71a5701ff6d6" />

That concludes the Step by Step guide on how to set up a custom policy to enforce application installation across VMs and Scaleset. Custom policy that I used and additional details of the custom policy can be found under the [Policy](https://github.com/pranabpm/VM-Application-deployment-with-Azure-Policy/tree/main/Policy) directory.





