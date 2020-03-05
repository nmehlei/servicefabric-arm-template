# servicefabric-arm-template
Example of ARM template for creating an Azure Service Fabric instance with dedicated premium data disks

# Add Data Disks to Template
See: https://github.com/nmehlei/servicefabric-arm-template/blob/master/template.json#L598-L608
```
"dataDisks": [
	{
		"diskSizeGB": 128,
		"lun": 0,
		"createOption": "Empty",
		"caching": "ReadOnly",
		"managedDisk": {
			"storageAccountType": "Premium_LRS"
		}
	}
]
```

# Add Powershell Script to initialize new Data Disk
See: https://github.com/nmehlei/servicefabric-arm-template/blob/master/PrepareDisk.ps1
```
Get-Disk |
    Where PartitionStyle -eq 'Raw' |
    Select-Object -First 1 |
    Initialize-Disk -PartitionStyle MBR -PassThru |
    New-Partition -DriveLetter F -UseMaximumSize |
    Format-Volume -FileSystem NTFS -NewFileSystemLabel "Data" -Confirm:$false
```

This script has to be uploaded to a publicly accessible azure blob storage account.

# Schedule invocation of Powershell Script
See: https://github.com/nmehlei/servicefabric-arm-template/blob/master/template.json#L438-L456
```
{
	"name": "[concat(parameters('vmNodeType0Name'),'_PrepareDisk')]",
	"properties": {
		"publisher": "Microsoft.Compute",
		"type": "CustomScriptExtension",
		"typeHandlerVersion": "1.9",
		"autoUpgradeMinorVersion": true,
		"settings": {
			"fileUris": [
				"[concat(parameters('scriptLocation'),'/PrepareDisk.ps1')]"
			]
		},
		"protectedSettings": {
			"commandToExecute": "powershell -ExecutionPolicy Unrestricted -File PrepareDisk.ps1",
			"storageAccountName": "[parameters('scriptStorageAccount')]",
			"storageAccountKey": "[parameters('scriptStorageAccountKey')]"
		}
	}
},
```

# Move Service Fabric Config/Storage Directory to new Data Disk
See: https://github.com/nmehlei/servicefabric-arm-template/blob/master/template.json#L467-L478
```
"settings": {
	...
	"nodeTypeRef": "[parameters('vmNodeType0Name')]",
	"dataPath": "F:\\\\SvcFab",
	"durabilityLevel": "Silver",
	...
},
```

