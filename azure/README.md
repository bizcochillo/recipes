Renew secrets of an App Registration

```console
az ad sp reset-credentials --name "APP_REGISTRATION_NAME" --years 2
```

Looking for a blob in a Storage Account with other data than prefix or tag requires to perform a Full-Search on names 

```powershell
$resourceGroupName="<resource_group_name>"
$storageAccName="<storage_account_name>"
$storageAcc=Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccName
$ctx=$storageAcc.Context
Get-AzStorageBlob -Context $ctx -Container $containerName | Where-Object {$_.Name -like "*<test_to_search_for>*"} 
```
