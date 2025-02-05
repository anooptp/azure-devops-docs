---
title: Add and configure a report server
titleSuffix: Azure DevOps Server
description: Learn how to add and configure a report server so you can include Azure DevOps reports to help you manage your software development projects.
ms.assetid: 1BB64B5F-708B-4E6E-9E88-4DCAEE58A231
ms.technology: devops-analytics
ms.topic: conceptual
ms.author: kaelli
author: KathrynEE
monikerRange: '< azure-devops' 
ms.date: 09/23/2021
---

# Add a report server

[!INCLUDE [temp](../includes/tfs-report-platform-version.md)]

Here's the first task in the four-task sequence to add reports to a team project. If you have already installed SQL Reporting Services and Analysis Server, then go to step 2 to upload reports.

[![Add a report server](media/step-1-add-a-report-server.png)](add-a-report-server.md)
[![Upload reports](media/step-2-upload-reports.png)](upload-reports.md)
[![Grant permissions](media/step-3-grant-permissions.png)](grant-permissions-to-reports.md) 
[![Review team activities](media/step-4-review-team-activities.png)](review-team-activities-for-useful-reports.md)


> [!IMPORTANT]
> You can only add a report server to an on-premises Azure DevOps Server, formerly named Team Foundation Server. If you're using Azure DevOps Services, adding a report server isn't a supported option, instead, you can use the [Analytics Service](../powerbi/what-is-analytics.md?toc=/azure/devops/report/toc.json&bc=/azure/devops/report/breadcrumb/toc.json). For Azure DevOps 2019, review 



On-premises installations of Azure DevOps can include reports to help you manage your software development projects. However, you need a report server as part of your deployment to use them. If you don't have one, you can add SQL Server Reporting Services to your deployment. 

> [!NOTE]  
> If you installed TFS on a client operating system, such as Windows 7, you can't add reporting as described here, because it isn't supported on client operating systems. 

## What do I need to know before adding a report server?  
<!-- BEGINSECTION class="md-qanda" -->

### Q: What kind of report server can I add?

**A:** You'll need to add SQL Server Reporting Services to provide a report server for TFS. Add Reporting Services to the SQL Server instance that is supporting TFS, or to a different instance.

### Q: What permissions do I need?

**A:** You need to be a member of the local administrators group on the server, the Team Foundation Administrators group in TFS, and the sysadmin group in SQL Server, or have the equivalent permissions. If you don't have them, [get those permissions now](/azure/devops/server/admin/add-administrator).

### Q: Can I add reporting if TFS is installed on a client operating system?
**A:** No. You can't add reporting and analysis services on a client operating system. Your choices aren't easy. If you still want to add reporting, do one of these options:
- [Move the team project collection](/azure/devops/server/admin/move-project-collection) to a deployment of TFS that supports reporting.
- [Restore your entire deployment to a different server](/azure/devops/server/admin/backup/tut-single-svr-home) that is running a server operating system and a version of SQL Server that supports reporting.

### Q: How do I find out if I already have reporting available for TFS?

**A:** Look in the administration console for Team Foundation Server. If you see a reporting node and it shows that reporting is configured and enabled, it's already there.  

![If this node isn't present, you don't have reports](media/tfs-admin-console-reporting.png)  

Jump to the bottom of this article to confirm that reports are available for the team project collection, and then move on to [Upload reports to a team project](upload-reports.md).

### Q: Do I need to back up my data first?
**A:** You should have a recent backup available. If you don't, you can [make a backup by using Scheduled Backups](/azure/devops/server/admin/backup/config-backup-sched-plan).

<!-- ENDSECTION -->


## Verify your options and upgrade if necessary

1.	Find out what version and edition of SQL Server you have by following the instructions provided in [Validate a SQL Server Installation](/sql/database-engine/install-windows/validate-a-sql-server-installation). 
	If the answer is SQL Server Express, keep following these steps. If it's any other edition, but you don't see any version and edition information about reporting or analysis services, jump ahead to [Add Reporting Services](#AddRSandAS). If you do see information about these services, then your SQL Server already has them installed, and you can jump ahead to [Add Reporting to TFS](#AddRStoTFS).  
2.	If your deployment is using SQL Server Express, [upgrade to a different edition](/sql/database-engine/install-windows/upgrade-to-a-different-edition-of-sql-server-setup). For more information about the editions that support reporting in TFS, see [Requirements and compatibility, SQL Server](/azure/devops/server/requirements#sql-server). 

<a id="AddRSandAS">  </a>

## Add Reporting Services and Analysis Services to SQL Server 

You've confirmed that you have an edition of SQL Server that supports reporting or you upgrade to one. Now add the reporting and analysis services features. You only have to do so if they aren't already configured.

1.	Open the SQL Server Installation Center to add features to an existing installation.  

	![Add, remove, and manage SQL Server features](media/sql-install-center.png)

	> [!NOTE]
    > For SQL Server 2017 or later versions, choose the **Multidimensional and Data Mining Mode** option during installation. To learn more, see [Manually install SQL Server](/azure/devops/server/install/sql-server/install-sql-server#install-sql-server).

	You might have to restart your computer during this process, particularly if you've finished upgrading from SQL Server Express.  

2.	Once your server has passed all the setup rules checks, pick the instance to add features to.   

	> [!TIP]
    > If you upgraded from SQL Server Express, you'll have a named instance called SQLEXPRESS.  

	![Add features to an existing instance](media/installation-type.png)  

3.	In **Feature Selection**, add these features:
	*	Analysis Services
	*	Reporting Services - Native
	*	Management Tools - Complete (not pictured)

	![Add features to your instance](media/feature-selection.png)

4.	Accept the defaults for service accounts in **Server Configuration**, unless your deployment has specific service account needs. Make sure all services are set to start automatically.  

	![Make sure services are set to automatic](media/service-accounts.png)  

5.	Add your user account in **Analysis Services Configuration**.  

6.	Choose **Install and Configure** for reporting services if you can; if not, choose **Install Only**.  

	If you're adding features after upgrading from SQL Server Express, Install Only might be the only option available.  

	![Install and configure if option is available](media/reporting-services-config.png)  

7. Finish the wizard, including any server restarts that might be required.

If you had to choose **Install Only** when you added reporting, you'll need to open Reporting Services Configuration Manager and do some more configuration. If not, jump ahead to [Add Reporting to TFS](#add_reporting_to_tfs).

## Configure Reporting Services Manually  

1.	Open Reporting Services Configuration Manager and connect to your report server.  

	![Make sure you include the instance name, if any](media/open-reporting-services-configuration-manager.png)  

2.	Once you've connected, make sure the service is started, then go to the **Web Service URL** page and apply all the defaults.  

3.	Now you'll need to create a database for reporting. On the **Report Server Database** page, the **Change Database** option will open the Report Server Database Configuration Wizard assist you in creating that database.  

 	![TFS needs this database for reporting](media/change-database.png)  

4.	When you specify the database server connection, include the instance name and the server name, separated by a backslash (`\`).  

	![Include the instance name, if any](media/report-database-server.png)  
	If you aren't using a named instance, you can just provide the server name.  

5.	Accept the default values on the rest of the pages of the wizard and wait for it to finish. It may take a few minutes.  

6.	Accept all the defaults on the **Report Manager URL** page.  

	![The fields are prepopulated; choose apply](media/report-manager-url.png)  

You've now fully configured reporting on the SQL Server instance that supports TFS. It's time to add the reporting features to TFS!  

<a id="add_reporting_to_tfs">  </a>  

<a id="AddRStoTFS">  </a>

## Add Reporting to TFS  

You'll need to add reporting in two places: to TFS itself, and then to your team project collection.

1.	Open the administration console for TFS and go to the **Reporting** node to start the configuration process.   

	![Edit the information to configure reporting](media/tfs-admin-reporting.png)  

2.	Select the **Use Reporting** check box. On the **Warehouse** tab, provide the name of the report server and instance for the warehouse database. Use the default name for the database (TFS_Warehouse), and test the connection.  

	![Use the default name for the database](media/reporting-warehouse.png)  

    > [!TIP]
    > Because you're configuring reporting for the first time, the databases don't exist yet. They will be created when you finish the configuration process.

3. Provide the same server and instance information for **Analysis Services**, but use the default name for the analysis database (TFS_Analysis). Provide an account name and password for a data sources account. It's a special account you've created just for this purpose as described in [Service accounts and dependencies in TFS](/azure/devops/server/admin/service-accounts-dependencies). 

	![You'll need to add the reader account information](media/analysis-services.png)  

    > [!TIP]
    > Test the connection before continuing.  

4.	For **Reports**, provide the server name and the instance name in **Server**, separated by a slash.  

	Use **Populate URLs** to automatically populate the next two boxes. Add the same data sources account information you provided on the previous tab.  

	![Provide the server, instance, and account info](media/reports-server.png)  
	Once you finish, databases will be created, and the warehouse will start building.  

	![Status information for configured services](media/config-report-status.png)  

5.	Now add the report server to the team project collection. Edit the information in **Reports Folder**.  

	![Edit the folder location to start configuring](media/team-project-collections.png)  

    > [!TIP]
    > If you don't see the **Reports Folder** tab, you might need to refresh or restart the administration console.  
 
6.	Unless you have reasons otherwise, use the example provided, and make sure you include the collection name.  

	![Include the collection name in the path](media/default-folder-location.png)

Now that you've added reporting to the server and to the team project collection, you're ready to start adding reports to your projects.

## Next step

[Upload reports to a team project](upload-reports.md) 

## Related content
You can learn more about installing and configuring SQL Server for use with on-premises TFS from these articles:  
*  [Supported versions and edition](/azure/devops/server/requirements#sql-server)  
*  [Working with named instances](/azure/devops/server/install/sql-server/install-sql-server)  
*  [Using SQL Server 2012 AlwaysOn](/azure/devops/server/install/sql-server/use-always-on-groups)  
*  [Understanding how TFS, SQL Server, and Reporting all work together](/azure/devops/server/architecture/sql-server-databases)   

### Set permissions
- [Set administrator permissions](/azure/devops/server/admin/add-administrator)  


### Change the data reader account  
- [Change the service account or password for SQL Server Reporting](/azure/devops/server/admin/change-service-account-or-password-sql-reporting)
- [Service accounts and dependencies in Team Foundation Server](/azure/devops/server/admin/service-accounts-dependencies)