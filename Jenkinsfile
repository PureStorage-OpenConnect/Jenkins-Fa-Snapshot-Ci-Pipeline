properties([
    parameters([
        string(name: 'Database'       , defaultValue: 'tpch-no-compression'),
        string(name: 'SourceInstance' , defaultValue: 'Z-STN-WIN2016-A\\DEVOPSPRD'),
        string(name: 'DestInstance'   , defaultValue: 'Z-STN-WIN2016-A\\DEVOPSDEV1'),
        string(name: 'PfaEndpoint'    , defaultValue: '10.225.112.10'),              
        string(name: 'SqlPackagePath' , defaultValue: 'C:\\SSDTTools\\Microsoft.Data.Tools.Msbuild\\lib\\net46\\sqlpackage.exe')              
  ])
])

node {
    stage('git checkout'){
        timeout(time:1, unit:'MINUTES') {
            checkout scm
        }
    }

    stage('Build Dacpac from SQLProj') {
        timeout(time:5, unit:'MINUTES') {
            bat "\"${tool name: 'Default', type: 'msbuild'}\" /p:Configuration=Release"
            stash includes: 'Jenkins-Fa-Snapshot-Ci-Pipeline\\bin\\Release\\Jenkins-Fa-Snapshot-Ci-Pipeline.dacpac', name: 'theDacpac'
        }
    }
    
    stage('Refresh test from production')
    {
        timeout(time:5, unit:'MINUTES') {
            withCredentials([string(credentialsId: 'PfaCredentialsFile', variable: 'PfaCredentialsFile'),
                             string(credentialsId: 'PfaUser'           , variable: 'PfaUser')]) {
                powershell 'Import-Module -Name PureStorageDbaTools; ' + 
                           '\$Pwd = Get-Content ' + "${PfaCredentialsFile}" + ' | ConvertTo-SecureString;'  +
                           '\$Creds = New-Object System.Management.Automation.PSCredential(\"' + "${PfaUser}" + '\",\$Pwd); ' +
                           'Invoke-PfaDbRefresh -RefreshDatabase ' + "${params.Database}"       + 
                                              ' -RefreshSource   ' + "${params.SourceInstance}" + 
                                              ' -DestSqlInstance ' + "${params.DestInstance}"   + 
                                              ' -PfaEndpoint     ' + "${params.PfaEndpoint}"    + 
                                              ' -PfaCredentials  \$Creds'
            }
        }
    }

    stage('Deploy Dacpac to SQL Server')
    {
        timeout(time:2, unit:'MINUTES') {
            unstash 'theDacpac'
            def ConnString = "server=${params.DestInstance};database=${params.Database}"
            bat "\"${SqlPackagePath}\" /Action:Publish /SourceFile:\"Jenkins-Fa-Snapshot-Ci-Pipeline\\bin\\Release\\Jenkins-Fa-Snapshot-Ci-Pipeline.dacpac\" /TargetConnectionString:\"${ConnString}\""
        }        
    }
}
