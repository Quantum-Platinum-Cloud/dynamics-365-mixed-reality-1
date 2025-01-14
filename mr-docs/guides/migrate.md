---
author: melissahellmund
description: This topic explains how to migrate Microsoft Dynamics 365 Guides content from one Microsoft Dataverse instance to another by using the Dynamics 365 Guides Content Migration Tool (Public Preview)
ms.author: mehellmu
ms.date: 04/28/2023
ms.topic: how-to
title: Migrate Dynamics 365 Guides content from one Microsoft Dataverse instance to another using the Content Migration Tool (Public Preview)
ms.reviewer: v-wendysmith
---

# Migrate Dynamics 365 Guides content from one Microsoft Dataverse instance to another using the Content Migration Tool (Public Preview)

The Content Migration Tool (Public Preview) for Microsoft Dynamics 365 Guides is a [Windows PowerShell](/windows-server/administration/windows-commands/powershell) script that copies content from one instance of Microsoft Dataverse to another. You can migrate all the content in an instance or just specific types of content (for example, 3D objects, images, and videos). After the migration is completed, the content exists in both instances. The content in the source instance isn't changed in any way.

The tool connects to two Microsoft Dataverse instances at the same time. It reads the content in the source instance and writes it to the destination instance. The instances can be part of the same tenant, or they can be in different tenants.

The tool supports migration between commercial and/or Microsoft Government Community Cloud (GCC) environments.

> [!IMPORTANT]
> By using the Content Migration Tool (Public Preview) for Dynamics 365 Guides, you acknowledge and accept all the limitations of using a preview tool. For example, you accept that the terms and commitments for Dynamics 365 Guides and other Microsoft commercial services and products don't apply to the Content Migration Tool. You also assume all risks that are associated with migrating your data between Dynamics 365 tenants. These risks include but aren't limited to the risk of different security, compliance, and privacy commitments, the risk of damage to or loss of data, and the risk of unavailability or interruption of the tool.

The tool supports the following types of migration.

| Type of migration | What is migrated? |
|-------------------|-------------------|
| All content | Everything that is stored in the instance that is active. (Telemetry events that are associated with a guide are excluded.) |
| Guides and associated content | Active guides, and any 3D objects, images, or videos that are related to those guides. Inactive content that is associated with a guide is also migrated. You can migrate all existing guides at one time or select specific guides. |
| All 3D objects, images, and videos | All active 3D objects, images, and videos that are stored in the instance. |
| Only 3D objects | Active 3D objects only. You can migrate all existing 3D objects at one time, all 3D objects that belong to a 3D object collection, or select specific 3D objects. |
| Only images | Active images only. You can migrate all existing images at one time or select specific images. |
| Only videos | Active videos only. You can migrate all existing videos at one time or select specific videos. |
| Only Object Anchors | Active Azure Object Anchors only. You can migrate all existing object anchors at one time or select specific object anchors. |

> [!NOTE]
> - If you have created a folder structure in your instance, [migrate it using Power Platform's export/import functionality](/dynamics365/mixed-reality/guides/admin-export-import-folders). Moving the folders first, and then the content ensures that the folder and content hierarchy is retained. 
> - [Website and Power Apps links](pc-app-website-powerapps-link.md) are migrated when you migrate guides. However, for Power Apps, manually import the apps into the new instance and update the links in the appropriate steps.
> - The Content Migration Tool doesn't support migration of guides that are based on schema v3 or v4. If you've updated your Dynamics 365 Guides solution to the latest version, but you still have a guide that is based on schema v3 or v4, [do a manual upgrade](./upgrade.md).
> - For 3D objects that belong to a 3D object collection, the parent 3D object won't be migrated unless you have the **System Admin** role.

## Prerequisites

- A system admin for the computer that you use to run the Windows PowerShell script.

- [Windows PowerShell](/powershell/scripting/install/installing-windows-powershell#how-to-check-the-version-of-powershell) version 5.1.18362.752 or later.

- Both the source instance and the destination instance uses Dynamics 365 Guides solution version 700.0.0.x or later.

- Both the source instance and the destination instance have the same major and minor version numbers.

- Credentials to sign in to the source instance in the **System Admin** role (preferable) or **Operator/Author** role (required).

- Credentials to sign in to the destination instance in the **System Admin** role (preferable) or **Author** role (required).

We also highly recommend that you [back up the contents](/power-platform/admin/backup-restore-environments#create-a-manual-backup) of both instances before you start the migration process.

## Download the tool and run the script

1. Download the Guides [Content Migration Tool](https://aka.ms/ContentMigrationTool). The zipped folder is downloaded to your downloads folder.

1. In Windows File Explorer, select the zipped folder, right-click and then select **Properties**.

1. In the **Properties** dialog box, on the **General** tab, select the **Unblock** check box, and then select **Apply**.

   :::image type="content" source="media/migration-unblock.PNG" alt-text="Unblock check box and Apply button":::

1. Select the zipped folder and extract the contents.

1. Run Windows PowerShell [as an administrator](/powershell/scripting/windows-powershell/starting-windows-powershell#how-to-start-windows-powershell-on-earlier-versions-of-windows). Select **Start**, type **PowerShell**, choose **Windows PowerShell**, and then select **Run as administrator**.

1. Navigate to the **ContentMigrationTool\_D365GuidesR2\_v700** \> **MigrationToolScripts** folder in the extracted archive.

1. In the **User Account Control** message box, select **Yes** to allow Windows PowerShell to make changes to your device.

1. Enter **Set-ExecutionPolicy RemoteSigned**, and then select **Y** to accept the change in execution policy.

    ![Execution Policy Change page.](media/migration-set-execution-policy.PNG "Execution Policy Change page")

1. Enter **.\ContentMigrationTool\_PublicPreview\_Dynamics365Guides.ps1**, and then select **Enter**.

    > [!NOTE]
    > Be sure to include the period at the beginning of the string.

1. Select any key to continue.

1. In the **PowerShell Interactive Login** dialog box, connect to the source instance by following these steps:

    1. In the **Deployment Type** field group, select the **Microsoft 365** option.

    1. Select the **Display list of available organizations** check box.

    1. Select the **Show Advanced** check box.

    1. In the **Online Region** field, select the online region for your instance. If your environment is in a GCC region, select **North America 2**. If you don't know the region, select **Don't know**.

    1. In the **User Name** and **Password** fields, enter the user name and password for the Microsoft Dataverse instance that you want to connect to.

    1. Select **Login**. If you don't select an organization to connect to, the tool connects to the last organization that was successful for both the source instance and the destination instance. If there is only one instance in the tenant, Windows PowerShell automatically connects to the default instance. In this case, skip to step 12.

        ![PowerShell Interactive Login dialog box configured for the source instance.](media/migration-powershell-login-1.PNG "PowerShell Interactive Login dialog box configured for the source instance")

    1. In the **Starting Login Process** dialog box, select the source instance (organization) to connect to.

        ![Source instance selected in the Starting Login Process dialog box.](media/migration-source-instance.PNG "Source instance selected in the Starting Login Process dialog box")

    1. Select **Login**.

1. In the **PowerShell Interactive Login** dialog box, connect to the destination instance by following these steps:

    1. In the **Deployment Type** field group, select the **Microsoft 365** option.

    1. Select the **Display list of available organizations** check box.

    1. Select the **Show Advanced** check box.

    1. In the **Online Region** field, select the online region for your instance. If your environment is in a GCC region, select **North America 2**. If you don't know the region, select **Don't know**.

    1. In the **User Name** and **Password** fields, enter the user name and password for the Microsoft Dataverse instance that you want to connect to.

    1. Select **Login**.

        ![PowerShell Interactive Login dialog box configured for the destination instance.](media/migration-powershell-login-2.PNG "PowerShell Interactive Login dialog box configured for the destination instance")

    1. In the **Starting Login Process** dialog box, select the destination instance (organization) to connect to.

        ![Destination instance selected in the Starting Login Process dialog box.](media/migration-destination-instance.PNG "Destination instance selected in the Starting Login Process dialog box")

    1. Select **Login**.

1. On the **Main Menu** page, select an option by entering the number that corresponds to it, and then select **Enter**.

    ![Main Menu page.](media/migration-main-menu.PNG "Main Menu page")

    > [!TIP]
    > In numbered lists where you can select more than one option, such as the numbered list on the **Select Guides to Migrate** page, separate the numbers for your selections with a comma, and then select **Enter**.
    > 
    > ![Selecting multiple options on the Select Guides to Migrate page.](media/migration-numbered-list.PNG "Selecting multiple options on the Select Guides to Migrate page")

1. When you're prompted to confirm your selection, enter **Y**, and then select **Enter**. To cancel the action, enter **N**, and then select **Enter**.

    ![Confirming your selection.](media/migration-choice-confirmation.PNG "Confirming your selection")

## See also

- [Export and import your folder structure between environments](admin-export-import-folders.md)
- [Troubleshoot moving content](migrate-troubleshoot.md)


[!INCLUDE[footer-include](../includes/footer-banner.md)]
