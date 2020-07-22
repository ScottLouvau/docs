The XForm.exe http server can serve remote requests.
**Make sure to do this only for data that anyone should have access to**; for user restricted access, host XForm in IIS.

To enable remote access, run elevated:

    netsh http add urlacl url="http://+:5073/" user="DOMAIN\USER"
    netsh advfirewall firewall add rule name="XForm 5073" dir=in action=allow protocol=TCP localport=5073 profile=domain

Afterward, 'XForm http' will detect it can bind the port for remote access and do so.

To disable remote access, run:

    netsh http delete urlacl url="http://+:5073/"
    netsh advfirewall firewall delete rule name="XForm 5073"


## To host in IIS

On the Server:
  - Open an elevated prompt on the Web Server.
  - Run "PowerShell -File [XFormEnlistment]\XForm\XForm.IIS\Install.IIS.ps1"
  - Create a database root folder [ex: C:\Production\XFormProduction]
  - Create a folder for the WebSite [ex: C:\Production\XForm.IIS]
  - Create a file share to the WebSite you can deploy to [ex: \\WebServer\XForm.IIS]

On a development machine:
  - Create a copy of XForm\XForm.IIS\Web.config. [ex: C:\NotEnlistment\XForm.Production.web.config]
     - Set the XFormProduction folder to your production folder.
     - In the authorization section, add the Windows users and groups you want to have access.
  - Run Build.cmd
  - Run Deploy.IIS.cmd \\WebServer\XForm.IIS C:\NotEnlistment\XForm.Production.web.config

On the Server:
  - Run inetmgr
  - Right-click on Sites and pick 'Add WebSite...'
  - Set the physical path to your website path [C:\Production\XForm.IIS]
  - Set the Binding type to 'https'
  - Choose an installed SSL certificate
  - Click OK
  - Open a browser and browse to https://localhost; verify the site loads.
  - Add data to your XForm database root. Verify tables appear in site IntelliSense.
  - Ensure the firewall is open for the port your site is running on and test remote access.
