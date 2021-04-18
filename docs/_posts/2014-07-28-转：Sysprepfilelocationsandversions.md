---
layout:     post
title:      转：Sysprepfilelocationsandversions
date:       2014-07-28
tags: [cnblogs]
---
在p2v一台windows server 2003时，发现报这样一个错误：

"Unable to locate the required Sysprep files. Upload them under 'C:\ProgramData\VMware\VMware vCenter Converter Standalone\sysprep\svr2003' on the Converter server machine. See 'Help' for more details.", -->       path = "C:\ProgramData\VMware\VMware vCenter Converter Standalone\sysprep\svr2003"

找到这样一篇文章：Sysprep file locations and versions 

http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=1005593

#### Symptoms

<li>
When attempting to customize the deployment of a virtual machine, the radio buttons are disabled (grayed out).
</li>
<li>
When a virtual machine is deployed from a template, the SID is always the same, despite the fact that you chose the option to generate a new SID during template deployment and guest operating system customization.
</li>
<li>
When attempting to create a new virtual machine from a template in vCenter Server,you see the error:Warning: Windows customization resources were not found on this server

</li>
<li>
In the guestcust.log file, you see the error:
 
deploy doesn't contain known sysprep files

</li>
<li>
When you ignore the errors, the virtual machine deployment does not get customized

</li>


#### Purpose

#### Resolution

1. Open a Windows command prompt. For more information, see [Opening a command or shell prompt (1003892)](http://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=1003892).
1. Change to the directory where the .exe file is saved.
1. Enter the name of the .exe file with the /x switch to extract the files. For example:WindowsServer2003-KB926028-v2-x86-ENU.exe /x 
1. When prompted, choose a directory for the extracted files.
1. Browse the directory and double-click the deploy.cab file.**Note**: In some cases, the deploy.cab file may be located within one of the subfolders created in Step 3.
1. Select and copy all the files to the Sysprep Directory and insert them into the correct folder. For example, if you are using Windows 2003, insert the files to the Windows 2003 folder found located at **directory_path**\svr2003.


1. Log in to the vCenter Server as an Administrator.
<li>
Click **Start** > **Programs** > **Accessories** > **Windows Explorer**.

</li>
<li>
Navigate to the Sysprep Directory as listed in the table below.

</li>
<li>
Right-click on the Sysprep .exe file and choose **Properties**.

</li>
<li>
Click the **Version** tab. Record the number at the top next to **File Version:**.

</li>


<li>
If vCenter Server is installed on Windows Server 2008 and above, **directory_path** is%ALLUSERSPROFILE%\VMware\VMware VirtualCenter\Sysprep which generally translates toC:\ProgramData\VMware\VMware VirtualCenter\Sysprep by default.Note: C:\ProgramData may be a hidden folder.

</li>
- If vCenter Server is installed on any other Windows operating system, **directory_path** is%ALLUSERSPROFILE%\Application Data\VMware\VMware VirtualCenter\Sysprep\ which generally translates toC:\Documents and Settings\All Users\Application Data\VMware\VMware VirtualCenter\Sysprep\ by default.
- To check the SID of a server deployed from a template, you can use the PsGetSid. For more information, see[http://technet.microsoft.com/en-us/sysinternals/bb897417](http://technet.microsoft.com/en-us/sysinternals/bb897417).   ****Note**: The preceding link was correct as of July 18, 2013. If you find the link is broken, provide feedback and a VMware employee will update the link.**

|**Windows Version**|**Sysprep Directory**|**Sysprep Version**
|Windows 2000 Server SP4 with Update Rollup 1The updated Deployment Tools are available in the Support\Tools\Deploy.cab file on the Windows 2000 SP4 CD-ROM|**directory_path**\2k|5.0.2195.2104
|Windows XP Pro SP2Download at[http://www.microsoft.com/en-us/download/details.aspx?id=18976](http://www.microsoft.com/en-us/download/details.aspx?id=18976)|**directory_path**\xp|5.1.2600.2180
|Windows 2003 Server SP1Download at[http://www.microsoft.com/en-us/download/details.aspx?id=23096](http://www.microsoft.com/en-us/download/details.aspx?id=23096)|**directory_path**\svr2003|5.2.3790.1830(srv03_sp1_rtm.050324-1447)
|Windows 2003 Server SP2Download at[http://www.microsoft.com/en-us/download/details.aspx?id=14830](http://www.microsoft.com/en-us/download/details.aspx?id=14830)|**directory_path**\svr2003|5.2.3790.3959(srv03_sp2_rtm.070216-1710)
|Windows 2003 Server R2Download at[http://www.microsoft.com/en-us/download/details.aspx?id=14830](http://www.microsoft.com/en-us/download/details.aspx?id=14830)|**directory_path**\svr2003|5.2.3790.3959(srv03_sp2_rtm.070216-1710)
|Windows 2003 x64Download at[http://www.microsoft.com/en-us/download/details.aspx?id=8287](http://www.microsoft.com/en-us/download/details.aspx?id=8287)|**directory_path**\svr2003-64|5.2.3790.3959(srv03_sp2_rtm.070216-1710)
|Windows XP x64Download at[http://www.microsoft.com/en-us/download/details.aspx?id=8287](http://www.microsoft.com/en-us/download/details.aspx?id=8287)|**directory_path**\xp-64|5.2.3790.3959(srv03_sp2_rtm.070216-1710)
|Windows XP Pro SP3Download at[http://www.microsoft.com/en-us/download/details.aspx?id=11282](http://www.microsoft.com/en-us/download/details.aspx?id=11282)|**directory_path**\xp|5.1.2600.5512
|Windows VistaSystem Preparation tools are built into the Windows Vista operating system and do not have to be downloaded.|Not Applicable|Not Applicable
|Windows Server 2008System Preparation tools are built into the Windows Server 2008 operating system and do not have to be downloaded.|Not Applicable|Not Applicable
|Windows Server 2008 R2System Preparation tools are built into the Windows Server 2008 R2 operating system and do not have to be downloaded.|Not Applicable|Not Applicable
|Windows 7System Preparation tools are built into the Windows 7 operating system and do not have to be downloaded.|Not Applicable|Not Applicable
|Windows 8System Preparation tools are built into the Windows 8 operating system and do not have to be downloaded.|Not Applicable|Not Applicable
|Windows Server 2012System Preparation tools are built into the Windows Server 2012 operating system and do not have to be downloaded.|Not Applicable|Not Applicable
|vCenter Server Virtual Appliance 5.x|/etc/vmware-vpx/sysprep/|Not Applicable

#### Additional Information

<li>
The [Basic System Administration](http://www.vmware.com/support/pubs/) guide, which contains further information regarding the installation of SysPrep tools

</li>
<li>
The **Operating System Compatibility for vSphere Client, vCenter Server, and VMware vCenter Update Manager** table in the [vSphere Compatibility Matrixes](http://www.vmware.com/pdf/vsphere4/r40/vsp_compatibility_matrix.pdf) for a list of supported operating systems for virtual image customization.

</li>
- [Troubleshooting template deployment or cloning when it fails (1004050)](http://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=1004050), which provides troubleshooting information for template deployment or cloning
- [VMware VirtualCenter TemplatesUsage and Best Practices, which provides for best practices on creating templates for vCenter Server](http://www.vmware.com/pdf/vc_2_templates_usage_best_practices_wp.pdf)
- For troubleshooting customization issues with Windows 8 and Windows Server 2012, see [Guest OS customization of Windows 8 and Windows Server 2012 may not complete (2037366)](http://kb.vmware.com/selfservice/search.do?cmd=displayKC&docType=kc&docTypeID=DT_KB_1_1&externalId=2037366).

