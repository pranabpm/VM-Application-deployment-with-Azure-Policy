Understanding the policy details
```
"if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "anyOf": [
            {
              "field": "Microsoft.Compute/virtualMachines/osProfile.windowsConfiguration",
              "exists": true
            },
            {
              "field": "Microsoft.Compute/virtualMachines/storageProfile.osDisk.osType",
              "like": "Windows*"
            },
            {
              "field": "Microsoft.Compute/imagePublisher",
              "in": [
                "MicrosoftVisualStudio",
                "MicrosoftWindowsDesktop",
                "MicrosoftWindowsServer"
              ]
            }
          ]
        }
      ]
    },
```

In this section of the policy I am running the policy on all windows machine. Depending on your specific scenario you may have other prerequisit for your own application that you can add to the "if" segment and filter down the machines to only those where you wish to install your application. 

```
"then": {
      "effect": "deployIfNotExists",
      "details": {
        "type": "Microsoft.Compute/virtualMachines",
        "name": "[field('name')]",
        "existenceCondition": {
          "allOf": [
            {
              "count": {
                "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*]",
                "where": {
                  "field": "Microsoft.Compute/virtualMachines/applicationProfile.galleryApplications[*].packageReferenceId",
                  "equals": "/subscriptions/XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXX/resourceGroups/Quick-POC/providers/Microsoft.Compute/galleries/vmgalleries/applications/Chrome/versions/1.0.0"
                }
              },
              "greater": 0
            }
          ]
        },
```
This section refers to the policy "deployIfNotExists" thats primary goal is to deploy an application if the state of VM doesn't match the goal. We are specifying the service type "virtualMachine" that we want to evaluate. For each virtual machine, we are evaluating the statement and when each statement under "allOf" element must be true. To evaluate it, we examine the galleryApplications array under the applicationProfile property on the VM. For each array element the field we evaluate is packageReferenceId. Here is a quick way to resource ID via PowerShell.

```
Get-AzGalleryApplicationVersion -GalleryName "vmgalleries" -ResourceGroupName "Quick-POC" -GalleryApplicationName "Chrome" | select-object Id
```
The output should look some thing similar to:

```
Id
--
/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXX/resourceGroups/Quick-POC/providers/Microsoft.Compute/galleries/vmgalleries/applications/Chrome/versions/1.0.0
```

from the output you will use the resourceId of the VM Application you want to ensure is installed which should look similar to:

```
"equals": "/subscriptions/XXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXX/resourceGroups/Quick-POC/providers/Microsoft.Compute/galleries/vmgalleries/applications/Chrome/versions/1.0.0"
```
Next is the role definition

```
"roleDefinitionIds": [
            "/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635"
          ],
```
You can pick and select the most approprete roleDefinition you need. The roleDefinition I am using is for full access. Full list of roleDefinition can be found [here](https://learn.microsoft.com/en-us/azure/role-based-access-control/built-in-roles) and you are welcome to create your own roleDefinitions.

```
"resources": [
                  {
                    "apiVersion": "2021-07-01",
                    "type": "Microsoft.Compute/virtualMachines/VMapplications",
                    "name": "[concat(parameters('vmName'), '/Chrome')]",
                    "location": "[parameters('location')]",
                    "properties": {
                      "packageReferenceId": "/subscriptions/7d84e459-664d-4e40-8b69-70001ccac7a3/resourceGroups/Quick-POC/providers/Microsoft.Compute/galleries/vmgalleries/applications/Chrome/versions/1.0.0"
                    }
                  }
                ]
```
In this case, we’re saying to add a new VMApplication with a packageReferenceId equal to that desired above. So, what this policy really does is – if the application isn’t installed on the VM – then put it there.


