---
title: "Microsoft Gets in on the Browser Hijacking Game"
modified_date: "2020-02-12"
last_modified_at: "2020-02-12"
---
**Update:** [Microsoft has reversed their decision](https://techcommunity.microsoft.com/t5/office-365-blog/update-to-microsoft-search-in-bing-through-office-365-proplus/ba-p/1161030) to automatically install the Microsoft Search in Bing extension. The extension will still be made available but will not be automatically deployed with Office 365 ProPlus. The original post continues below.

---

Starting next month, Microsoft plans to use Office 365 ProPlus to push a browser extension for Google Chrome that will [change users' default search engines to Bing](https://docs.microsoft.com/en-us/deployoffice/microsoft-search-bing). Version 2002 of Office 365 ProPlus will forcibly install the [Microsoft Search in Bing](https://chrome.google.com/webstore/detail/microsoft-search-in-bing/obdappnhkfoejojnmcohppfnoeagadna) extension for all Chrome users who do not already use Bing as their default search engine.

Understandably, [many system administrators are frustrated with the announcement](https://www.reddit.com/r/sysadmin/comments/es6xp5/office_365_proplus_to_change_chromes_default/), as unwanted browser extensions that change end-user settings are usually considered malware and are blocked accordingly. In fact, Microsoft's own security tools already block dozens of programs that exhibit similar behavior.

On GitHub, [users are responding to the change](https://github.com/MicrosoftDocs/OfficeDocs-DeployOffice/issues) by opening issues in the [OfficeDocs-DeployOffice repository](https://github.com/MicrosoftDocs/OfficeDocs-DeployOffice). So far, it does not appear that Microsoft has responded to this influx of unsolicited feedback outside of publishing a [blog post extolling the virtues of Bing](https://techcommunity.microsoft.com/t5/office-365-blog/introducing-and-managing-microsoft-search-in-bing-through-office/ba-p/1110974).

## Who Is Affected?

At this point, only businesses that have deployed Office 365 ProPlus are affected. Depending on the organization's Office 365 license, ProPlus is the version of Office delivered to end-users when they install Office from the [office.com](https://www.office.com/) portal. According to Microsoft, not all Office 365 plans include the ProPlus version of Office:
> This extension is included only with Office 365 ProPlus. It isn’t included with Office 365 Business, which is the version of Office that comes with certain business plans, such as the Microsoft 365 Business plan and the Office 365 Business Premium plan.

## Firefox Is Next

According to Microsoft, a similar extension for Firefox is also on the way:
> Support for the Firefox web browser is planned for a later date. We will keep you informed about support for Firefox through the Microsoft 365 Admin Center and [this article](https://docs.microsoft.com/en-us/deployoffice/microsoft-search-bing).

## Removing The Extension

By making the extension an opt-out feature, Microsoft is putting the onus on system administrators to deploy a method for blocking its installation. While there are [official ways to prevent the extension from being installed](https://docs.microsoft.com/en-us/deployoffice/microsoft-search-bing#how-to-exclude-the-extension-for-microsoft-search-in-bing-from-being-installed), there is no easy Microsoft-supported method for removing the extension once it has already been deployed. Instead, Microsoft recommends running the following command as an administrator on each affected machine using a script:
```
C:\Program Files (x86)\Microsoft\DefaultPackPC\MainBootStrap.exe uninstallAll
```

It should also be possible to blacklist the extension with the 3rd party Group Policy templates for Chrome and Firefox provided by Google and Mozilla.

Unfortunately, Group Policy and other enterprise management tools do not always apply to BYOD devices, leaving users who install Office on their personal machines with little recourse except to notice and remove the extension on their own if they find it undesirable.

---

**References:**
- [Microsoft Search in Bing and Office 365 ProPlus - Microsoft Docs](https://docs.microsoft.com/en-us/deployoffice/microsoft-search-bing)
- [Microsoft to force Chrome default search to Bing using Office 365 installer - The Verge](https://www.theverge.com/2020/1/22/21077280/microsoft-chrome-bing-extension-office-365-proplus-installer-default-search-engine)
- [Microsoft to forcibly install Bing search extension in Chrome for Office 365 ProPlus users - ZDNet](https://www.zdnet.com/article/microsoft-to-forcibly-install-bing-search-extension-in-chrome-for-office-365-proplus-users/)
- [Microsoft to Force Bing Search in Chrome for Office 365 ProPlus Users - BleepingComputer](https://www.bleepingcomputer.com/news/microsoft/microsoft-to-force-bing-search-in-chrome-for-office-365-proplus-users/)
