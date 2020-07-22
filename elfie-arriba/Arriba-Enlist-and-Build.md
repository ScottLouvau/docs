### Install Dependencies
+ Install Visual Studio 2017 [Community, Enterprise]
+ Install Node.js [https://nodejs.org/en/]

### Enlist in the elfie-arriba Repo  
`git clone https://www.github.com/Microsoft/elfie-arriba`

### Get Arriba.Web Dependencies from Node
In Directory elfie-arriba\Arriba\Arriba.Web:  
`npm install`  

### Build Arriba
The Arriba build has two parts - building the Arriba engine and building the Arriba.Web website.  
+ The engine builds Arriba.dll, Arriba.Server.exe (a console app service host), and Arriba.IIS (an IIS-hostable service host).  
+ The website build copies a Configuration folder into Arriba.Web\configuration and then creates a JavaScript "bundle" which contains the site logic along with your configuration (your theme color, site name, custom details views, and so on).

To Build the Arriba Engine:  
`elfie-arriba\Arriba\Build.cmd`

To Build the Arriba.Web website:  
`elfie-arriba\Arriba\Arriba.Web\node build.js <configFolderPath>`

Run 'node build.js configuration.default' to build the default, unthemed website, which will call the Arriba service on the same machine as the site.

You can copy the folder Arriba\Arriba.Web\configuration.default (or one of the Arriba\Databases\*\Configuration folders) and then customize it to change the site name and theme color, default listing columns and sort orders, details view shown per table, feedback e-mail addresses, access denied message, sample queries, and more. See the comments in [onfiguration.default\Configuration.jsx](https://github.com/Microsoft/elfie-arriba/blob/master/Arriba/Arriba.Web/configuration.default/Configuration.jsx) to see what can be configured.