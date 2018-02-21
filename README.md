# Jenkins Fa Snapshot Ci Pipeline Gallery

## Jenkinsfile.simple
This is a build pipeline written in Jenkins script format that performs the following functions:

1. Checks out a SQL Server data tools project from a local GIT repo, this is created by cloning this repository to C:\Projects
2. Builds the project into a DACPAC, in order to do this you will need DACFX ( https://www.microsoft.com/en-us/download/details.aspx?id=55114 ) or an installation of Visual Studio (community edition will suffice) installed for "Data Storage and Processing" workloads.
3. Refreshes a test database from production, the pre-requisites for this are for the database names to be the same and each database should reside on its own Pure Storage FlashArray volume with the both the 'Production' and 'Test' volumes being equal in size. The dbatools and PureStoragePowerShellSDK modules need to be installed first, these can be found in the PowerShell gallery, finally install the Refresh-Dev-PsFunc PowerShell function from the PureStorage SQL Scripts repo, ensuring that this is autoloaded from the a PoweerShell profile that is picked up by the windows account that the Jenkins service runs under.
4. Deploys the DACPAC to the refreshed test database.


