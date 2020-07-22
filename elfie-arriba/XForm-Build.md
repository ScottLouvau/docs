To Build XForm:
  - Install Visual Studio 2017 Community+ [Version 15.3.5+]
  - Install the Visual Studio workload 'Desktop development with C++' and component 'Windows 10 SDK (10.0.15063.0) for Desktop'. [Run 'Visual Studio Installer' to add features to VS]
  - Install NPM from https://nodejs.org/en/download/
  - In XForm.Web, run 'npm install'
  - In XForm, run '..\\.nuget\NuGet.exe restore XForm.sln'
  - In XForm, run 'Build.cmd'
  - The build outputs are placed in XForm\bin\x64\Release.