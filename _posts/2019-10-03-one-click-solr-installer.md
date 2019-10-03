---
title: One Click Solr Installer
categories:
- blog
excerpt: Install Solr as a Windows service with one click!
tags:
- solr
- installer
- sitecore
- powershell
date: 2019-10-03T04:00:00.000+00:00
header:
  overlay_image: assets\images\blog\2019-10-03-one-click-solr-installer\hero.png
  overlay_filter: "0.5"
  show_overlay_title: true
---

# About
One click Solr Installer is a creation which came from the desire of wanting to spend less time installing stuff and more time doing stuff. After getting inspired by [Jeremy Davis' Low-Effort Solr Installs](https://jermdavis.wordpress.com/2017/10/30/low-effort-solr-installs/) post and using his script, I strived to make things even simpler. Yes, opening the script and configuring all the parameters was too much work for me.
``` powershell
Param(
    $installFolder = "c:\solr",
    $nssmVersion = "2.24",
    $solrVersion = "6.6.2",
    $solrPort = "8983",
    $solrHost = "localhost",
    $solrSSL = $true,
    $JREVersion = "1.8.0_151"
)
```

## Download and Run!
My script is available on GitHub: [https://github.com/kimmminjae/one-click-solr-installer](https://github.com/kimmminjae/one-click-solr-installer)

Download both files then run ```one-click-solr-installer.bat```. You will need Admin privileges.
# Modifications

## JRE Version and Validation
Configuring the JREVersion was the biggest pain point since JRE version isn't something I keep in the top of my head.
``` powershell
try {
    $javaVerbose = java -verbose
    cls
    $javaVersionPath = ($javaVerbose -split '\n')[0]
    $javaVersionPath = $javaVersionPath -replace '\\lib\\rt.jar]' -replace ''
    $javaVersionPath = $javaVersionPath -replace '\[Opened ' -replace ''
    if ($javaVersionPath -Match 'x86') {
        Write-Host 'JRE (64-bit) not installed. Please install JRE (64-bit). Download: https://www.java.com/en/download/manual.jsp'
        Write-Host 'Press any key to exit...'
        $null = $Host.UI.RawUI.ReadKey('NoEcho,IncludeKeyDown');
        exit
    }
}
catch {
    Write-Host 'JRE (64-bit) not installed. Please install JRE (64-bit). Download: https://www.java.com/en/download/manual.jsp'
    Write-Host 'Press any key to exit...'
    $null = $Host.UI.RawUI.ReadKey('NoEcho,IncludeKeyDown');
    exit
}
```
I realized that I can get the Java version by calling the ``` java -verbose``` command. This returned the location of where JRE was installed.
```
[Opened C:\Program Files\Java\jre1.8.0_221\lib\rt.jar]
```
With some string trimming, I was able to extract the path. I also added some validations on the way to make sure the user had JRE 64-bit installed. This validation might prevent some headaches later when installing Sitecore.

## Listing All the Available Solr Versions and Taking the Input
Before this project, I had multiple scripts with predefined versions embedded in the scripts. I then used the appropriate script for the version I wanted. I didn't like that. First, I made some changes to let the user input the version they wanted. Then I improved it by crawling the solr archive site to retrieve all the available versions of Solr. Even the latest ones.
``` powershell
# Retrieve the list of Solr versions
$WebResponseObj = Invoke-WebRequest -Uri "https://archive.apache.org/dist/lucene/solr/" -UseBasicParsing
$innerHTMLofLinks = $WebResponseObj.links | select outerHTML
$innerHTMLs = $innerHTMLofLinks -split '\n'
$outItems = New-Object System.Collections.Generic.List[System.String]
foreach ($innerHTML in $innerHTMLs) {
    $innerHTML = $innerHTML -replace '\@\{outerHTML.*/\"\>' -replace ''
    $innerHTML = $innerHTML -replace '\/\<\/a\>\}' -replace ''
    if($innerHTML -match '([0-9].[0-9]?[0-9].?[0-9]?-?[A-B]?\w*)') {
    $version = $innerHTML
    $outItems.Add($version)
    }
}
$versionsString = ""
$itemCount = 1
foreach ($version in $outItems) {
    if ($itemCount % 4) {
        $versionsString = $versionsString + $version + "`t `t"
    } else {
        $versionsString = $versionsString + $version + "`n"    
    }
    $itemCount++
}
Write-Host $versionsString

Write-Host "`nAbove are all the available versions of Solr."
$versionInput = Read-Host -Prompt "Please enter the desired Solr version"
$isValidInput = $outItems -ccontains $versionInput
while (!$isValidInput) {
    Write-Host "Your input is invalid."
    $versionInput = Read-Host -Prompt "Please enter the desired Solr version (eg. 7.2.1)"
    $isValidInput = $outItems -ccontains $versionInput
}
```
![](/assets/images/blog/2019-10-03-one-click-solr-installer/solr-installer-version-selection.png)
## Configuration
Okay, I lied. It's not a ONE CLICK Installer. You need to type more than once to get Solr installed. I didn't plan to add the configuration setting when I was planning for this project, and that's why I called it One-Click.
``` powershell
$solrVersion = $versionInput
$solrVersionText = $solrVersion -replace '\.' -replace ''
$installFolder = "c:\solr" + $solrVersionText
$solrHost = "solr" + $solrVersionText
Write-Host ' '
Write-Host ' '
Write-Host 'Host Name Options:'
Write-Host '1.' $solrHost
Write-Host '2. localhost'
Write-Host '3. Custom Input'
$confirmation = Read-Host 'Choose (1/2/3)'
if ($confirmation -eq '3') {
    $solrHost = Read-Host 'Input the desired Host Name: '
} elseif ($confirmation -eq '2') {
    $solrHost = 'localhost'
}
Write-Host $solrHost 'selected'
Write-Host ' '
Write-Host ' '
Write-Host 'Default Installation Folder: ' $installFolder
$confirmation2 = Read-Host 'Use Default? (Y/N)'
if ($confirmation2 -eq 'n') {
    $installFolder = Read-Host 'Input the desired installation location'
}
Write-Host 'Installation Folder:' $installFolder
```
I still tried my best to make it simple. The default values are still there and just pressing 2 enter keys would bypass this. I wanted to let people choose their host name and installation folder if they wanted to.
![](/assets/images/blog/2019-10-03-one-click-solr-installer/solr-installer-config-selection.png)