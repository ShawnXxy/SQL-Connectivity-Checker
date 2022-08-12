# Azure MySQL Connectivity Checker

This PowerShell script will run some connectivity checks from this machine to the server and database.  
- Supports Single, Flexible (please provide FQDN).
- Supports Public Cloud (*.msyql.database.azure.com), Azure China (*.mysql.database.chinacloudapi.cn)  

**In order to run it you need to:**
1. Ensure MySQL .Net Connector is installed. Please refer to https://dev.mysql.com/downloads/connector/net/8.0.html
2. Open Windows PowerShell ISE (in Administrator mode if possible)
In order for a network trace to be collected along with the tests ('CollectNetworkTrace' parameter), PowerShell must be run as an administrator.

3. Open a New Script window

4. Paste the following in the script window:

```powershell
[System.Reflection.Assembly]::LoadWithPartialName("MySql.Data")
$parameters = @{
    # Supports Single, Flexible (please provide FQDN, priavete endpoint and Vnet Ingested Flexible is supported)
    # Supports Public Cloud (*.msyql.database.azure.com), Azure China (*.mysql.database.chinacloudapi.cn)
    Server = '.mysql.database.azure.com' # or any other supported FQDN
    Database = ''  # Set the name of the database you wish to test, 'information_schema' will be used by default if nothing is set
    User = ''  # Set the login username you wish to use, 'AzMySQLConnCheckerUser' will be used by default if nothing is set
    Password = ''  # Set the login password you wish to use, 'AzMySQLConnCheckerPassword' will be used by default if nothing is set

    ## Optional parameters (default values will be used if omitted)
    SendAnonymousUsageData = $true  # Set as $true (default) or $false
    RunAdvancedConnectivityPolicyTests = $true  # Set as $true (default) or $false, this will load the library from Microsoft's GitHub repository needed for running advanced connectivity tests
    ConnectionAttempts = 1 # Number of connection attempts while running advanced connectivity tests
    DelayBetweenConnections = 1 # Number of seconds to wait between connection attempts while running advanced connectivity tests
    CollectNetworkTrace = $true  # Set as $true (default) or $false
    #EncryptionProtocol = '' # Supported values: 'Tls 1.0', 'Tls 1.1', 'Tls 1.2'; Without this parameter operating system will choose the best protocol to use
}

$ProgressPreference = "SilentlyContinue";
if ("AzureKudu" -eq $env:DOTNET_CLI_TELEMETRY_PROFILE) {
    $scriptFile = '/ReducedMySQLConnectivityChecker.ps1'
} else {
    $scriptFile = '/AzureMySQLConnectivityChecker.ps1'
}
$scriptUrlBase = 'https://raw.githubusercontent.com/ShawnXxy/AzMySQL-Connectivity-Checker/xixia'
cls
Write-Host 'Trying to download the script file from GitHub (https://github.com/ShawnXxy/AzMySQL-Connectivity-Checker), please wait...'
try {
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 -bor [Net.SecurityProtocolType]::Tls11 -bor [Net.SecurityProtocolType]::Tls
    Invoke-Command -ScriptBlock ([Scriptblock]::Create((Invoke-WebRequest ($scriptUrlBase + $scriptFile) -UseBasicParsing -TimeoutSec 60).Content)) -ArgumentList $parameters
    }
catch {
    Write-Host 'ERROR: The script file could not be downloaded:' -ForegroundColor Red
    $_.Exception
    Write-Host 'Confirm this machine can access https://github.com/ShawnXxy/AzMySQL-Connectivity-Checker/' -ForegroundColor Yellow
    Write-Host 'or use a machine with Internet access to see how to run this from machines without Internet. See how at https://github.com/ShawnXxy/AzMySQL-Connectivity-Checker/' -ForegroundColor Yellow
}
#end
```
5. Set the parameters on the script. You must set the server name and database name. User and password are optional, but best practices.

6. Run it.  
   Results are displayed in the output window. If the user has permissions to create folders, a folder with the resulting log file will be created, along with a ZIP file (`AllFiles.zip`). When running on Windows, the folder opens automatically after the script completes.

7. Examine the output for any issues detected, and recommended steps to resolve the issue.

## Run from Linux

With the current release, PowerShell uses .NET 5.0 as its runtime. PowerShell runs on Windows, macOS, and Linux platforms.  

In order to run this script on Linux you need to 
1. Installing PowerShell on Linux (if you haven't before).
   See how to get the packages at https://docs.microsoft.com/powershell/scripting/install/installing-powershell-core-on-linux

2. Installing Mono on Linux (if you haven't before).    
   See how to get the packages at https://www.mono-project.com/download/stable/#download-lin-ubuntu

3. Download MySQL .Net connector and save on Linux from https://dev.mysql.com/downloads/connector/net/
    ![image](/rsc/mysql-net-connector.png)
    Making sure the folder is unzipped.

4. After the package is installed, run ***pwsh*** from a Linux terminal. In the openned PowerShell console, register mysqlclient library by running the following command:

    ```powershell
    sudo gacutil -i "{path_to_folder_from_step#3}/v4.xx/MySql.Data.dll"  # replace {path_to_folder_from_step#3} with the path to the folder where the package is saved on your Linux machine
    
    ```

5. Set the parameters on the following script then copy paste it to the terminal. You must set the server name and database name. User and password are optional, but best practices.

```powershell
[system.reflection.Assembly]::LoadFrom("{path_to_folder_from_step#3}/v4.xx/MySql.Data.dll") # replace {path_to_folder_from_step#3} with the path to the folder where the package is saved on your Linux machine
$parameters = @{
    # Supports Single, Flexible (please provide FQDN, priavete endpoint and Vnet Ingested Flexible is supported)
    # Supports Public Cloud (*.msyql.database.azure.com), Azure China (*.mysql.database.chinacloudapi.cn)
    Server = '.mysql.database.azure.com' # or any other supported FQDN
    Database = ''  # Set the name of the database you wish to test, 'information_schema' will be used by default if nothing is set
    User = ''  # Set the login username you wish to use, 'AzMySQLConnCheckerUser' will be used by default if nothing is set
    Password = ''  # Set the login password you wish to use, 'AzMySQLConnCheckerPassword' will be used by default if nothing is set

    ## Optional parameters (default values will be used if omitted)
    SendAnonymousUsageData = $true  # Set as $true (default) or $false
    RunAdvancedConnectivityPolicyTests = $true  # Set as $true (default) or $false, this will load the library from Microsoft's GitHub repository needed for running advanced connectivity tests
    ConnectionAttempts = 1 # Number of connection attempts while running advanced connectivity tests
    DelayBetweenConnections = 1 # Number of seconds to wait between connection attempts while running advanced connectivity tests
    CollectNetworkTrace = $true  # Set as $true (default) or $false
    #EncryptionProtocol = '' # Supported values: 'Tls 1.0', 'Tls 1.1', 'Tls 1.2'; Without this parameter operating system will choose the best protocol to use
}

$ProgressPreference = "SilentlyContinue";
if ("AzureKudu" -eq $env:DOTNET_CLI_TELEMETRY_PROFILE) {
    $scriptFile = '/ReducedMySQLConnectivityChecker.ps1'
} else {
    $scriptFile = '/AzureMySQLConnectivityChecker.ps1'
}
$scriptUrlBase = 'https://raw.githubusercontent.com/ShawnXxy/SQL-Connectivity-Checker/xixia'
cls
Write-Host 'Trying to download the script file from GitHub (https://github.com/ShawnXxy/SQL-Connectivity-Checker), please wait...'
try {
    [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 -bor [Net.SecurityProtocolType]::Tls11 -bor [Net.SecurityProtocolType]::Tls
    Invoke-Command -ScriptBlock ([Scriptblock]::Create((Invoke-WebRequest ($scriptUrlBase + $scriptFile) -UseBasicParsing -TimeoutSec 60).Content)) -ArgumentList $parameters
    }
catch {
    Write-Host 'ERROR: The script file could not be downloaded:' -ForegroundColor Red
    $_.Exception
    Write-Host 'Confirm this machine can access https://github.com/ShawnXxy/SQL-Connectivity-Checker/' -ForegroundColor Yellow
    Write-Host 'or use a machine with Internet access to see how to run this from machines without Internet. See how at https://github.com/ShawnXxy/SQL-Connectivity-Checker/' -ForegroundColor Yellow
}
#end
```
5. Examine the output for any issues detected, and recommended steps to resolve the issue.

## How to run this from machines whithout Internet access

**In order to run it from machines without Internet access you need to:**

1. From a machine with Internet access
    - Navigate to https://github.com/ShawnXxy/SQL-Connectivity-Checker
    - Click on the green button named 'Clone or download'
    - Select 'Download ZIP'

1. Copy the 'SQL-Connectivity-Checker-master.zip' file to the machine you need to run tests from.

1. Extract all the files into a folder.

1. Open Windows PowerShell ISE in Administrator mode.  
For the better results, our recommendation is to use the advanced connectivity tests which demand to start PowerShell in Administrator mode. You can still run the basic tests, in case you decide not to run this way. Please note that script parameters 'RunAdvancedConnectivityPolicyTests' and 'CollectNetworkTrace' will only work if the admin privileges are granted.

1. From PowerShell ISE, open the file named 'RunLocally.ps1' you can find in the previous folder.

1. Set the parameters on the script, you need to set server name. Database name, user and password are optional but desirable.

1. Save the changes.

1. Click Run Script (play button). You cannot run this partially or copy paste to the command line.

1. The results can be seen in the output window.
If the user has the permissions to create folders, a folder with the resulting log file will be created.
When running on Windows, the folder will be opened automatically after the script completes.
A zip file with all the log files (AllFiles.zip) will be created.

<!-- ## Running SQL Connectivity Checker in containerized environment

In order to troubleshoot your containerized application you'll have to temporarily deploy a Powershell Image which will allow you to execute this script and collect the results, you can see all the available Powershell Images [here](https://hub.docker.com/_/microsoft-powershell).

Our suggestion would be to use a lightweight image for this purpose, such as `lts-alpine-3.10` image.

### Kubernetes

The following steps show the Kubernetes kubectl commands required to download the image and start an interactive PowerShell session.

```
kubectl run -it sqlconncheckerpowershellinstance --image=mcr.microsoft.com/powershell:lts-alpine-3.10
```

The following command is used to exit the current Powershell session.
```
exit
```

The following command is used to attach to an existing Powershell instance.
```
kubectl attach -it sqlconncheckerpowershellinstance
```

The following command is used to delete the pod running this image when you no longer need it.

```
kubectl delete pod sqlconncheckerpowershellinstance
```

### Docker

The following steps show the Docker commands required to download the image and start an interactive PowerShell session.

```
docker run -it --name sqlconncheckerpowershellinstance --image=mcr.microsoft.com/powershell:lts-alpine-3.10
```

The following command is used to exit the current Powershell session.
```
exit
```

The following command is used to attach to an existing Powershell instance.
```
docker attach sqlconncheckerpowershellinstance
```

The following command is used to delete the container running this image when you no longer need it.

```
docker container rm sqlconncheckerpowershellinstance
``` -->


# Contributing

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
