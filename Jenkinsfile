properties([
    parameters([
        string(name: 'Database'       , defaultValue: 'tpch-no-compression'),
        string(name: 'SourceInstance' , defaultValue: 'Z-STN-WIN2016-A\\DEVOPSPRD'),
        string(name: 'DestInstance'   , defaultValue: 'Z-STN-WIN2016-A\\DEVOPSDEV1'),
        string(name: 'CredentialsFile', defaultValue: 'C:\\Temp\\Secure-Credentials.txt'),              
        string(name: 'PfaEndpoint'    , defaultValue: '10.225.112.10')              
  ])
])

node {
    stage('git checkout'){
        timeout(time:1, unit:'MINUTES') {
            git checkout
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
            def stdout = powershell(returnStdout: true, script: '''
                             \$Pwd   = Get-Content '${params.CredentialsFile}' | ConvertTo-SecureString
                             \$Creds = New-Object System.Management.Automation.PSCredential (\"pureuser\", \$pwd)
                             Invoke-PfaDbRefresh -RefreshDatabase ${params.Database}       `
                                                 -RefreshSource   ${params.SourceInstance} `
                                                 -DestSqlInstance ${params.DestInstance}   `
                                                 -PfaEndpoint     ${params.PfaEndpoint}    `
                                                 -PfaCredentials  $Creds 
                                    ''')
                println stdout
        }
    }

    stage('Deploy Dacpac to SQL Server')
    {
        timeout(time:2, unit:'MINUTES') {
            unstash 'theDacpac'
            def ConnString = "server=${params.DestInstance};database=${params.Database}"
            bat "\"C:\\Program Files (x86)\\Microsoft SQL Server\\140\\DAC\\bin\\sqlpackage.exe\" /Action:Publish /SourceFile:\"Jenkins-Fa-Snapshot-Ci-Pipeline\\bin\\Release\\Jenkins-Fa-Snapshot-Ci-Pipeline.dacpac\" /TargetConnectionString:\"${ConnString}\""
        }        
    }
}
