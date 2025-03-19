# Instruction

## Atifactory hot commands

- set nexus proxy for powershell gallery
```pwsh
Register-PSRepository -Name PSGallery -SourceLocation "http://nexus.local/repository/powershellgallery" -PublishLocation "http://nexus.local/repository/powershellgallery" -InstallationPolicy Trusted
```

- delete default PSGallery 
```pwsh
Unregister-PSRepository -Name Nexus
```

- nuget set proxy repo
```pwsh
dotnet nuget add source -n nexus http://nexus.local/repository/nuget-group/index.json --allow-insecure-connections --protocol-version 3
```

- nuget remove default repo
```pwsh
dotnet nuget remove source nuget
```

- choco add proxy repo
```pwsh
choco.exe source add -n=nexus -s=http://nexus.local/repository/chocolatey-group/
```
- choco revome default repo
```pwsh
choco.exe source remove -n=chocolatey
```
