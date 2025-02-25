---
uid: Granting_admin_consent
---

# Granting admin consent for Teams actions

When the Microsoft Teams Chat Integration functionality is used, users require admin consent for certain actions, e.g. to create teams or channels, or to add members to a team.

After you have granted this consent, you will need to configure the Microsoft tenant for your cloud-connected organization.

To grant admin consent and configure your tenant:

1. In the Admin app, check whether the correct organization is mentioned in the header bar.

1. If a different organization should be selected, click the organization selector in the top-right corner and select the organization in the list.

   ![Organization selector](~/user-guide/images/CloudAdmin_Selector.png)

1. In the sidebar on the left, go to *Organization* > *Overview*.

1. Click the *Grant Admin Consent* button.

   This will redirect you to Microsoft's admin consent endpoint to grant consent for the required permissions.

   ![Admin consent](~/user-guide/images/CloudAdmin_Admin_Consent.png)

   > [!NOTE]
   > In order to grant tenant-wide consent, you have to be an administrator for that tenant, e.g. Global Administrator.

1. Review the requested permissions, and if you agree, click the *Accept* button.

1. After clicking *Accept*, you will be redirected to a configuration screen where you can link your Microsoft tenant with your organization.

   ![Tenant configuration](~/user-guide/images/CloudAdmin_Tenant_Configuration.png)

1. Click *Confirm*.

1. In the pop-up window, log in to our multi-tenant app with your Microsoft account.

   After you log in successfully, your tenant is configured.

   > [!NOTE]
   > Make sure to log in with the same user as you granted consent with.
