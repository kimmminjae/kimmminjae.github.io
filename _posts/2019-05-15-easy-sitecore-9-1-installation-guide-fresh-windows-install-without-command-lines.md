---
date: 2019-05-16T03:02:35.000+00:00
header:
  overlay_image: "/assets/images/home-hero-background.jpg"
  overlay_filter: "0.5"
title: Easy Sitecore 9.1 Installation Guide
modified: ''
categories:
- blog
excerpt: Without Command Line, from a Fresh Windows Install
tags:
- Sitecore 9.1
- Sitecore Installation
- Installation Guide
- Sitecore
- Sitecore Experience Platform 9.1
- SIF-Less

---
# Relevant Links

### Sitecore 9.1.0 XP0 Single On Premises

##### Sitecore 9.1.0 XP0 installation media

[https://dev.sitecore.net/Downloads/Sitecore_Experience_Platform/91/Sitecore_Experience_Platform_91_Initial_Release.aspx](https://dev.sitecore.net/Downloads/Sitecore_Experience_Platform/91/Sitecore_Experience_Platform_91_Initial_Release.aspx "https://dev.sitecore.net/Downloads/Sitecore_Experience_Platform/91/Sitecore_Experience_Platform_91_Initial_Release.aspx")

##### Sitecore License

If you or your employer doesn't have a Sitecore License, you can get a 60 day trial license here: [https://www.sitecore.com/getting-started/developer-trial](https://www.sitecore.com/getting-started/developer-trial "Sitecore License Trial")

### SQL Server 2017 (or 2016 SP2 or Greater)

[https://www.microsoft.com/en-us/sql-server/sql-server-downloads](https://www.microsoft.com/en-us/sql-server/sql-server-downloads "SQL Server 2017")

### Java Runtime Environment (JRE)

[https://www.java.com/en/download/](https://www.java.com/en/download/ "Java Runtime Environment (JRE)")

### Low-Effort Solr Install

##### Jeremy Davis' blog post about his Low-Effort Solr Install script

[https://jermdavis.wordpress.com/2017/10/30/low-effort-solr-installs/](https://jermdavis.wordpress.com/2017/10/30/low-effort-solr-installs/ "Jeremy Davis' blog post about his Low-Effort Solr Install script")

##### Direct link to his code

[https://gist.github.com/jermdavis/8d8a79f680505f1074153f02f70b9105#file-install-solr-ps1](https://gist.github.com/jermdavis/8d8a79f680505f1074153f02f70b9105#file-install-solr-ps1 "Direct link to his code")

### Sitecore Installation Framework (SIF)

##### If you don't have SIF installed

    Register-PSRepository -Name SitecoreGallery -SourceLocation https://sitecore.myget.org/F/sc-powershell/api/v2
    
    Set-PSRepository -Name SitecoreGallery -InstallationPolicy Trusted
    
    Install-Module -Name SitecoreInstallFramework -Repository SitecoreGallery

##### If you have an older version of SIF

    Update-Module SitecoreInstallFramework

### SIF-Less

[http://www.rockpapersitecore.com/2019/03/sif-less-2-2-auto-update-and-ui-runner/](http://www.rockpapersitecore.com/2019/03/sif-less-2-2-auto-update-and-ui-runner/ "SIF-Less")