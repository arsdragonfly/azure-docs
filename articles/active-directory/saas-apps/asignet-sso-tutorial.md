---
title: 'Tutorial: Azure AD SSO integration with AsignetSSOIntegration'
description: Learn how to configure single sign-on between Azure Active Directory and AsignetSSOIntegration.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: CelesteDG
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 11/21/2022
ms.author: jeedes

---

# Tutorial: Azure AD SSO integration with AsignetSSOIntegration

In this tutorial, you'll learn how to integrate AsignetSSOIntegration with Azure Active Directory (Azure AD). When you integrate AsignetSSOIntegration with Azure AD, you can:

* Control in Azure AD who has access to AsignetSSOIntegration.
* Enable your users to be automatically signed-in to AsignetSSOIntegration with their Azure AD accounts.
* Manage your accounts in one central location - the Azure portal.

## Prerequisites

To get started, you need the following items:

* An Azure AD subscription. If you don't have a subscription, you can get a [free account](https://azure.microsoft.com/free/).
* AsignetSSOIntegration single sign-on (SSO) enabled subscription.
* Along with Cloud Application Administrator, Application Administrator can also add or manage applications in Azure AD.
For more information, see [Azure built-in roles](../roles/permissions-reference.md).

## Scenario description

In this tutorial, you configure and test Azure AD SSO in a test environment.

* AsignetSSOIntegration supports **SP** and **IDP** initiated SSO.

## Add AsignetSSOIntegration from the gallery

To configure the integration of AsignetSSOIntegration into Azure AD, you need to add AsignetSSOIntegration from the gallery to your list of managed SaaS apps.

1. Sign in to the Azure portal using either a work or school account, or a personal Microsoft account.
1. On the left navigation pane, select the **Azure Active Directory** service.
1. Navigate to **Enterprise Applications** and then select **All Applications**.
1. To add new application, select **New application**.
1. In the **Add from the gallery** section, type **AsignetSSOIntegration** in the search box.
1. Select **AsignetSSOIntegration** from results panel and then add the app. Wait a few seconds while the app is added to your tenant.

 Alternatively, you can also use the [Enterprise App Configuration Wizard](https://portal.office.com/AdminPortal/home?Q=Docs#/azureadappintegration). In this wizard, you can add an application to your tenant, add users/groups to the app, assign roles, as well as walk through the SSO configuration as well. [Learn more about Microsoft 365 wizards.](/microsoft-365/admin/misc/azure-ad-setup-guides)

## Configure and test Azure AD SSO for AsignetSSOIntegration

Configure and test Azure AD SSO with AsignetSSOIntegration using a test user called **B.Simon**. For SSO to work, you need to establish a link relationship between an Azure AD user and the related user in AsignetSSOIntegration.

To configure and test Azure AD SSO with AsignetSSOIntegration, perform the following steps:

1. **[Configure Azure AD SSO](#configure-azure-ad-sso)** - to enable your users to use this feature.
    1. **[Create an Azure AD test user](#create-an-azure-ad-test-user)** - to test Azure AD single sign-on with B.Simon.
    1. **[Assign the Azure AD test user](#assign-the-azure-ad-test-user)** - to enable B.Simon to use Azure AD single sign-on.
1. **[Configure AsignetSSOIntegration SSO](#configure-asignetssointegration-sso)** - to configure the single sign-on settings on application side.
    1. **[Create AsignetSSOIntegration test user](#create-asignetssointegration-test-user)** - to have a counterpart of B.Simon in AsignetSSOIntegration that is linked to the Azure AD representation of user.
1. **[Test SSO](#test-sso)** - to verify whether the configuration works.

## Configure Azure AD SSO

Follow these steps to enable Azure AD SSO in the Azure portal.

1. In the Azure portal, on the **AsignetSSOIntegration** application integration page, find the **Manage** section and select **single sign-on**.
1. On the **Select a single sign-on method** page, select **SAML**.
1. On the **Set up single sign-on with SAML** page, click the pencil icon for **Basic SAML Configuration** to edit the settings.

    ![Screenshot shows to edit Basic SAML Configuration.](common/edit-urls.png "Basic Configuration")

1. On the **Basic SAML Configuration** section, the user does not have to perform any step as the app is already pre-integrated with Azure.

1. Click **Set additional URLs** and perform the following step if you wish to configure the application in **SP** initiated mode:    

    In the **Sign-on URL** text box, type the URL:
    `https://trim.corp.microsoft.com/sso.ashx`

1. On the **Set-up single sign-on with SAML** page, in the **SAML Signing Certificate** section,  find **Federation Metadata XML** and select **Download** to download the certificate and save it on your computer.

    ![Screenshot shows the Certificate download link.](common/metadataxml.png "Certificate")

1. On the **Set up AsignetSSOIntegration** section, copy the appropriate URL(s) based on your requirement.

	![Screenshot shows to copy configuration appropriate URL.](common/copy-configuration-urls.png "Attributes")  

### Create an Azure AD test user

In this section, you'll create a test user in the Azure portal called B.Simon.

1. From the left pane in the Azure portal, select **Azure Active Directory**, select **Users**, and then select **All users**.
1. Select **New user** at the top of the screen.
1. In the **User** properties, follow these steps:
   1. In the **Name** field, enter `B.Simon`.  
   1. In the **User name** field, enter the username@companydomain.extension. For example, `B.Simon@contoso.com`.
   1. Select the **Show password** check box, and then write down the value that's displayed in the **Password** box.
   1. Click **Create**.

### Assign the Azure AD test user

In this section, you'll enable B.Simon to use Azure single sign-on by granting access to AsignetSSOIntegration.

1. In the Azure portal, select **Enterprise Applications**, and then select **All applications**.
1. In the applications list, select **AsignetSSOIntegration**.
1. In the app's overview page, find the **Manage** section and select **Users and groups**.
1. Select **Add user**, then select **Users and groups** in the **Add Assignment** dialog.
1. In the **Users and groups** dialog, select **B.Simon** from the Users list, then click the **Select** button at the bottom of the screen.
1. If you are expecting a role to be assigned to the users, you can select it from the **Select a role** dropdown. If no role has been set up for this app, you see "Default Access" role selected.
1. In the **Add Assignment** dialog, click the **Assign** button.

## Configure AsignetSSOIntegration SSO

To configure single sign-on on **AsignetSSOIntegration** side, you need to send the downloaded **Federation Metadata XML** and appropriate copied URLs from Azure portal to [AsignetSSOIntegration support team](mailto:us@asignet.com). They set this setting to have the SAML SSO connection set properly on both sides.

### Create AsignetSSOIntegration test user

In this section, you create a user called Britta Simon in AsignetSSOIntegration. Work with [AsignetSSOIntegration support team](mailto:us@asignet.com) to add the users in the AsignetSSOIntegration platform. Users must be created and activated before you use single sign-on.

## Test SSO 

In this section, you test your Azure AD single sign-on configuration with following options. 

#### SP initiated:

* Click on **Test this application** in Azure portal. This will redirect to AsignetSSOIntegration Sign-on URL where you can initiate the login flow.  

* Go to AsignetSSOIntegration Sign-on URL directly and initiate the login flow from there.

#### IDP initiated:

* Click on **Test this application** in Azure portal and you should be automatically signed in to the AsignetSSOIntegration for which you set up the SSO. 

You can also use Microsoft My Apps to test the application in any mode. When you click the AsignetSSOIntegration tile in the My Apps, if configured in SP mode you would be redirected to the application sign-on page for initiating the login flow and if configured in IDP mode, you should be automatically signed in to the AsignetSSOIntegration for which you set up the SSO. For more information about the My Apps, see [Introduction to the My Apps](../user-help/my-apps-portal-end-user-access.md).

## Next steps

Once you configure AsignetSSOIntegration you can enforce session control, which protects exfiltration and infiltration of your organization’s sensitive data in real time. Session control extends from Conditional Access. [Learn how to enforce session control with Microsoft Cloud App Security](/cloud-app-security/proxy-deployment-aad).