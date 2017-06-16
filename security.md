# Scope
This document will serve as a technical guideline for the setup of Let's Encrypt Free SSL certificates on Azure Web Apps. (https://letsencrypt.org)

# Let's Encrypt
Let's Encrypt is a free, automated, open Certificate authority sponsored and supported by a  number of big players in the industry including Cisco, Facebook, Mozilla and Chrome.
Just like with any other Certificate Authority, in order to be granted a certificate you need to be able to prove ownership or control over the domain in question. Further more Let's Encrypt will only issue a certificate that is valid for a period of 3 months. This is done so as to encourage the automated process of re-newing the certificate frequently and therefore enhance security. Let's Encrypt uses the ACME protocol (https://ietf-wg-acme.github.io/acme/) in order to achieve this.

Primarily Let's Encrypt was designed for Linux environments but as its popularity and reputation grew stronger a C# implementaion of the ACME protocol has been implemented (https://github.com/ebekker/ACMESharp). The way we install Let's Encrypt on our Azure Web App environments is by using a KUDU Web App Extension which utilizes this C# implementation of the ACME protocol.

## Checklist
* Azure Resource Group
* Azure Web App
    * Service Plan Standard or Above (with custom domains)
    * Custom domains set up
* Azure Storage Account
* Active Directory Application

## Step 1
* Grant access to Let's Encrypt on Web App
* Grant access to Let's Encrypt on Storage Account

## Step 2
We now need to set up a number of application settings on our web app.

![alt text](https://github.com/IncredibleWeb/architecture/blob/master/Img/Let's%20Encrypt.png)

* **letsencrypt:Tenant** - is highlighted in the above image as #1.
* **letsencrypt:SubscriptionId** - is #2
* **letsencrypt:ResourceGroupName** - is #3
* **letsencrypt:ClientId** - is the GUID from $app.ApplicationId
* **letsencrypt:ClientSecret** - is the value from $password
* **AzureWebJobsStorage** - Storage Account connection string
* **AzureWebJobsDashboard** - Storage Account connection string

## Step 3
Next, we have to install the Let's Encrypt Extension on our Web App. We may do this by accessing the KUDU dashboard on our Web App. The KUDU dashboard may be accessed by simply inserting **.scm** in your **.azurewebsites.net** url ex https://mysite.scm.azurewebsites.net. 
The extension can be installed via the Extensions Tab. 

**Note**: You have to restart your web app after installing the extension.

## Step 4
Once you run the extension, you will be prompted with a wizard-like setup which will guide you through the installation and setup of your SSL certificates. It is important to note that the Extension should be able to read your confugurations from the web app and extract them to the wizard.

**.Well-Known**
The extension will install a new folder called '.well-known' folder. This folder will contain files with hashkeys identifying your certificates. **The files within this folder should be able accessible**. Before you proceed with the certificate installation on any of the domains, make sure that you are able to access these files by URL. If you are not able to access these files neither is the extension and the Certificate installation will fail.

On **Express JS** the following line (or similar) should be added to the app.js in order to allow routing access to the static **.well-known** folder:
`app.use('/.well-known', express.static('.well-known'));`







