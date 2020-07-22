To Deploy Arriba on a server:
* Make a 'Production' folder and a file share for it.
* From an enlistment, in the Arriba folder, run Deploy.cmd <ServerName>

* Install IIS
    * Run an elevated Powershell prompt.
    * Enable PowerShell execution [Set-ExecutionPolicy Unrestricted; change this back afterward].
    * Install IIS Components [Import-Module Production\Configuration\InstallArriba.psm1; Install-IIS]
* Get a "Let's Encrypt" SSL certificate
    * Get a DNS name [Ex: Use changeip.com]
    * Run inetmgr elevated.
    * Add a website on port 80 serving inetpub\wwwroot bound to your specific DNS name.
    * In the Windows Firewall, allow Port 80 traffic through.
    * In your router configuration, allow Port 80 through to your server.
    * Get [LetsEncrypt-Win-Simple](https://github.com/Lone-Coder/letsencrypt-win-simple/releases)
    * From an elevated prompt, run LetsEncrypt.exe. It should detect the bound site, add a challenge response, get a certificate, and install it.
    * Configure the Arriba.IIS and Arriba.Web sites to use HTTPs and the certificate.
    * Allow them through the Firewall.
    * Build a version of Arriba.Web with your DNS name as the service url. (Ex: https://mydnsname.com:42785)
