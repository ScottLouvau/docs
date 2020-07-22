Bung is a configured version of Arriba.Web providing instant search for work items from Visual Studio Online, TFS, or other sources. 

This repository has:
- [Arriba.TfsWorkItemCrawler](https://github.com/Microsoft/elfie-arriba/tree/master/Arriba/Tools/Arriba.TfsWorkItemCrawler), run to get new and changed work items and index them.
- [Arriba.Web\parts.optional\WorkItemDetails](https://github.com/Microsoft/elfie-arriba/tree/master/Arriba/Arriba.Web/parts.optional/WorkItemDetails), with a default details view for WorkItems.
- [Databases\Louvau](https://github.com/Microsoft/elfie-arriba/tree/master/Arriba/Databases/Louvau), which contains an example configuration for a VSO database.

## Creating a Configuration
To index your own VSO database, you need to create a configuration. This tells Arriba where to crawl items from, who should have access, and how you want the website themed.

- Make a copy of the Arriba\Databases\Louvau folder for your configuration (ex: Arriba\Databases\YourProject).
- Edit config.json. Fix the arribaTable, itemCountLimit, owners, readers, and itemDatabase name for your VSO database.
- Edit Configuration\Configuration.jsx. Replace 'Louvau' and 'Bung' with your itemDatabase name. Fix the directLinkUrl.

## Run the first Crawl on your development machine
- In the Arriba folder, run "Build.cmd bung" to get the Crawler copied into the bin\Release folder.
- From an elevated prompt, run `Arriba\bin\Release\Arriba.Server.exe`.
- Run `Arriba\bin\Release\Arriba.TfsWorkItemCrawler <YourConfigName> -i`

## Build and Try the Website on your development machine
- From the Arriba.Web folder, run `node build.js ..\Databases\<YourConfigName>\Configuration`
- From Arriba.Web, run `npm start`
- Open the site and see how it looks (in Chrome or IE; Edge won't allow the localhost website to query the localhost service)

'build.js' copies your Configuration folder to Arriba.Web\Configuration, and then builds a JavaScript "bundle" with the website logic and your customizations and theming. It copies changes to the Arriba.Web\Configuration folder back to where they came from, so you can edit the copy in Arriba.Web directly without losing work. The Arriba.Web\Configuration folder will not be checked in by git, so you can't accidentally publish your configuration to GitHub.

'npm start' runs a development web server to serve the website as http://localhost:8080, and it will watch for changes to the website files and automatically update the site in the browser as they change. This means if you edit the configuration in Arriba.Web\Configuration, you can save the files and see changes immediately.

[[https://github.com/Microsoft/elfie-arriba/blob/master/images/Bung_StartScreen.PNG|alt=Bung Start Screen]]

## Finish configuring settings
- Finish editing Databases\<YourConfigName>\Configuration\Configuration.jsx. Set a good feedback alias, default columns, a good access denied request URL, and any default Grids you want.
- Finish editing Databases\<YourConfigName>\config.json. Set the readers to the same groups which have read access to your items in VSO itself [under the Settings gear in the VSO UI]. 

## Customize the Work Item Details view
- Copy [Arriba.Web\parts.optional\WorkItemDetails\WorkItemDetails.jsx](https://github.com/Microsoft/elfie-arriba/blob/master/Arriba/Arriba.Web/parts.optional/WorkItemDetails/WorkItemDetails.jsx) into Databases\<YourConfigName>\Configuration and rename it.
- In WorkItemDetails.jsx, replace "./" with "../parts.optional/WorkItemDetails/".
- In WorkItemDetails.jsx, replace "../../parts" with "../parts".
- Add your own JSX classes to abstract components, if desired.
- In Configuration.jsx, import your Details class and reference it in 'customDetailsProviders'
- From Arriba.Web, run "node build.js <YourConfigFolderPath>" to build Arriba.Web for your configuration.

[[https://github.com/Microsoft/elfie-arriba/blob/master/images/Bung_DetailsView.PNG|alt=Bung Details View]]

## Deployment Pre-Work
- Find an appropriate service account to crawl your VSO database and ensure it has permission to read from VSO via Azure Active Directory [AAD].
- Find a Windows Server 2012 R2+ server to host the site.
- Choose a name for the server and register a short name (CNAME) on your network if desired.
- Get an SSL certificate. Make sure to get the short and fully qualified forms of the physical machine name and CNAME.
- Create a Production folder and file share on your server which you can write to.

- Edit config.json to give crawler write access to the service account ("owners:"), and to set the arribaServiceUrl to your real service name (https://YourShortName:42785).
- Edit Configuration.jsx to set the service URL to your real service name.

## Deploy Your Instance to your server
- From Arriba, run:
```
Build.cmd bung
Arriba.Web\node build.js <YourConfigFolderPath>
Deploy.cmd <YourServerName> bung
```

- On the server, from PowerShell:
```
Import-Module "<ProductionFolderPath>\Configuration\InstallArriba.psm1"
Install-Arriba -ProductionFolder <ProductionFolderPath> -CertificateFilePath <YourSslCertificate>
```

This should install IIS with the required features, add an HTTP site to redirect to HTTPS, add an HTTPS site pointing to Arriba.Web, add an HTTPS service on port 42785 pointing to Arriba.IIS, and open the required firewall ports.

Note: Currently the Web Platform Installer step doesn't wait to complete. If it doesn't finish correctly, run the WebPlatformInstaller.msi (downloaded into Production\Configuration) manually and then run it and add the "URL Rewrite 2.0" component.

## Verify Crawling on your server
- On the server, run a command prompt as the service account.
- From Production\bin\Release:
- Run `Arriba.Csv /mode:setCreators /users:u:DOMAIN\ServiceAccount` to give your service account create permissions.
- Run: `Arriba.TfsWorkItemCrawler <YourDbName> -i` to get bugs (since last crawl, or all if no previous crawl)
- Verify no errors.
- Create a scheduled task to crawl every 15-60 minutes.
- Run the task and verify it succeeds.

If you see errors, verify that IIS_IUSRS can write to the Production\DiskCache folder, 'Windows Authentication' is enabled for Arriba.IIS, and your crawl account has table create permissions.