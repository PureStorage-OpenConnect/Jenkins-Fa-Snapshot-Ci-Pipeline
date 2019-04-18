# Jenkins-Fa-Snapshot-Ci-Pipeline
Jenkins Pipeline to illsutrate the use of a database refresh in a contunuous integration (CI) workflow.
## Overview

![pipeline](https://user-images.githubusercontent.com/15145995/53749355-1c7a2580-3e9f-11e9-83aa-7cacdae17bda.PNG)

This example Jenkins Pipeline checks a SQL Server data tools project and solution out of GitHub (the code), builds this into a DACPAC (the artifict), refreshes a pseudo test database from a pseudo production database and then applies the DACPAC to the test database.

## Prerequisites

1. The following software components need to be installed on the build server:
- Jenkins
- msbuild for SQL Server database targets,
- Data Tools framework (DAC Fx),
- PureStorageDbaTools PowerShell module, the installation of which will also install the dbatools and PureStoragePowerShellSDK.

2. The following plugins need to be installed on the Jenkins instance:
- Git 
- Pipeline
- msbuild 
- PowerShell

3. The user databases that act as the source and target of the refresh element of the pipeline require that:
 - their datafiles and the transaction log file reside on a single FlashArray volume,
 - their files reside on the same FlashArray.
    
 ### msbuild and SQL Server Data Tools Installation
 
 1. Download Visual Studio 2019 Community edition from this [link](https://visualstudio.microsoft.com/thank-you-downloading-visual-studio/?sku=Community&rel=16).
 
 2. Install Visual Studio via the executable downloaded in the previous step, ensure that the tool set for "Data storage and processing is installed:
 
 ![image](https://user-images.githubusercontent.com/15145995/56358336-3b761200-61d6-11e9-85bd-2325e4c81137.png)

 3. Downloasd the command line for nuget (nuget.exe) from this [link](https://dist.nuget.org/win-x86-commandline/v4.7.0/nuget.exe).
 
 4. Add the absolute path of nuget.exe to the PATH variable.
 
 5. Install SQL Server Data Tools:
 
    `nuget.exe install Microsoft.data.tools.msbuild -ExcludeVersion -OutputDirectory "C:\SSDTTools"`
    
 6. Configure the environment for SQL Server Data Tools by running the following three commands from within a DOS command shell window with Administrator privileges:

    `setx PATH "%PATH%;C:\SSDTTools\Microsoft.Data.Tools.Msbuild\lib\net46" /M`
    
    `setx SQLDBExtensionsRefPath C:\SSDTTools\Microsoft.Data.Tools.Msbuild\lib\net46 /M`
    
    `setx SSDTPath C:\SSDTTools\Microsoft.Data.Tools.Msbuild\lib\net46 /M`
    
 7. Open a DOS command shell whilst logged in as the domain account that the Azure DevOps build agent runs under. 
    Check that msbuild and sqlpackage can be found by using the following commands:

    `where msbuild`
    
    `where sqlpackage`
    
    If the tooling is installed correctly, the where commands will return the absolute paths for msbuild and sqlpackage.
 
 ### PureStorageDbaTools Installation
 
 1. Check that the PowerShell gallery is a trusted repository by running the following PowerShell function:

    `Get-PsRepository`
    
    This should return the following output:

    ![image](https://user-images.githubusercontent.com/15145995/52906043-2879ac80-323c-11e9-93b5-438acc7035c0.png)
    
 2. If the PowerShell gallery is not set up as a trusted repository, run the following command:

    `Register-PSRepository -Default -InstallationPolicy Trusted`
    
 3. Install the PureStorageDbaTools module, this will also install the dbatools and PureStoragePowerShellSDK modules:
 
    `Install-Module -Name PureStorageDbaTools`
 
 ### Pipeline Creation and Configuration
 
 1. At the top level of the Jenkins console navigate to "Global tools configuration".
 
 2. Under the section for MSBuild, click "Add MSBuild":

![image](https://user-images.githubusercontent.com/15145995/56358600-0a4a1180-61d7-11e9-98ce-37bbe63b0f65.png)

 3. In the 'Name' text box, enter the string Default.
 
 4. In the "Path to MSBuild2 text box, enter the string:

 C:\Program Files (x86)\Microsoft Visual Studio\2017\Community\MSBuild\15.0\Bin\msbuild.exe
 
 5. Ensure that the the Jenkins service (service as in Windows service) is running under a Windows domain account that can:

 - Connect to the database that is the source of the database refresh
 - Connect to the database that is the target of the database refresh
 - Offline the database that is the target of the database refresh
 -  Online the database that is the target of the database refresh
 
 for testing purposes, the simplest way to achieve this is to give the login the sysadmin privilege on the target instance.
 
 6. Navigate to Jenkins -> New Item enter a name in the text box under "Enter an item name" and then hit Pipeline.
 
 7. In the Pipeline section, select "Pipeline script from SCM" from the pulldown list of options and then GIT for the SCM 

![image](https://user-images.githubusercontent.com/15145995/56359212-d5d75500-61d8-11e9-87d1-b83bb2568ce6.png)

 8. In the "Repository URL" text box enter:

 https://github.com/PureStorage-OpenConnect/Jenkins-Fa-Snapshot-Ci-Pipeline
 
 9. Hit Apply followed by Save.
 
 10. 
 
 ### Instigating a Build
 
 1. 
 
 ![image](https://user-images.githubusercontent.com/15145995/56360186-c6a5d680-61db-11e9-89ee-29fef92892cc.png)
 
 
 
 
 
