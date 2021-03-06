---
title: Enable delegate access scenarios in an Outlook add-in
description: Briefly describes delegate access and discusses how to configure add-in support.
ms.topic: article
ms.date: 04/03/2019
localization_priority: Normal
---

# Enable delegate access scenarios in an Outlook add-in (preview)

A mailbox owner can use the delegate access feature to [allow someone else to manage their mail and calendar](https://support.office.com/article/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926). This article specifies which delegate permissions the Office JavaScript API supports and describes how to enable delegate access scenarios in your Outlook add-in.

> [!IMPORTANT]
> Delegate access for Outlook add-ins is currently [in preview](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/outlook-requirement-set-preview) and only supported in clients on Office build 16.0.10712.10000 or above and with their mailbox on Exchange Online. Delegate access APIs should not yet be used in production environments.

## Supported permissions for delegate access

The following table describes the delegate permissions that the Office JavaScript API supports.

|Permission|Value|Description|
|---|---:|---|
|Read|1 (000001)|Can read items.|
|Write|2 (000010)|Can create items.|
|DeleteOwn|4 (000100)|Can delete only the items they created.|
|DeleteAll|8 (001000)|Can delete any items.|
|EditOwn|16 (010000)|Can edit only the items they created.|
|EditAll|32 (100000)|Can edit any items.|

> [!NOTE]
> Currently the API supports getting existing delegate permissions, but not setting delegate permissions.

The [DelegatePermissions](/javascript/api/outlook/office.mailboxenums.delegatepermissions) object is implemented using a bitmask to indicate the delegate's permissions. Each position in the bitmask represents a particular permission and if it's set to `1` then the delegate has the respective permission. For example, if the second bit from the right is `1`, then the delegate has **Write** permission. You can see an example of how to check for a specific permission in the [Get delegate permissions](#get-delegate-permissions) section later in this article.

## Configure the manifest

To enable delegate access scenarios in your add-in, you must set the [SupportsSharedFolders](/office/dev/add-ins/reference/manifest/supportssharedfolders) element to `true` in the manifest under the parent element `DesktopFormFactor`. At present, other form factors are not supported.

> [!IMPORTANT]
> Because delegate access for Outlook add-ins is currently in preview, add-ins that use the `SupportSharedFolders` element cannot be published to AppSource or deployed via centralized deployment.

The following example shows the `SupportsSharedFolders` element set to `true` in a section of the manifest.

```XML
...
<VersionOverrides xmlns="http://schemas.microsoft.com/office/mailappversionoverrides" xsi:type="VersionOverridesV1_0">
  <VersionOverrides xmlns="http://schemas.microsoft.com/office/mailappversionoverrides/1.1" xsi:type="VersionOverridesV1_1">
    ...
    <Hosts>
      <Host xsi:type="MailHost">
        <DesktopFormFactor>
          <SupportsSharedFolders>true</SupportsSharedFolders>
          <FunctionFile resid="residDesktopFuncUrl" />
          <ExtensionPoint xsi:type="MessageReadCommandSurface">
            <!-- configure selected extension point -->
          </ExtensionPoint>

          <!-- You can define more than one ExtensionPoint element as needed -->

        </DesktopFormFactor>
      </Host>
    </Hosts>
    ...
  </VersionOverrides>
</VersionOverrides>
...
```

## Get delegate permissions

You can get an item's shared properties in Compose or Read mode by calling the [item.getSharedPropertiesAsync](/office/dev/add-ins/reference/objectmodel/preview-requirement-set/office.context.mailbox.item#getsharedpropertiesasyncoptions-callback) method. This returns a [SharedProperties](/javascript/api/outlook/office.sharedproperties) object that currently provides the delegate's permissions, the owner's email address, and the REST URL of the owner's mailbox.

The following example shows how to get the shared properties of a message or appointment and check if the delegate has **Write** permission.

```js
Office.context.mailbox.item.getSharedPropertiesAsync(callback);

function callback (asyncResult) {
  if (asyncResult.status === Office.AsyncResultStatus.Succeeded) {
    var sharedProperties = asyncResult.value;
    console.log(JSON.stringify(sharedProperties));

    var delegatePermissions = sharedProperties.delegatePermissions;

    // Determine if user can do the expected action.
    // E.g., do they have Write permission?
    if ((delegatePermissions & MailboxEnums.Write) != 0 ) {
      // Expected action...
    }
  }
}
```

## See also

- [Allow someone else to manage your mail and calendar](https://support.office.com/article/allow-someone-else-to-manage-your-mail-and-calendar-41c40c04-3bd1-4d22-963a-28eafec25926)
- [Calendar sharing in Office 365](https://support.office.com/article/calendar-sharing-in-office-365-b576ecc3-0945-4d75-85f1-5efafb8a37b4)
- [How to order manifest elements](/office/dev/add-ins/develop/manifest-element-ordering)
- [Mask (computing)](https://en.wikipedia.org/wiki/Mask_(computing))
- [JavaScript bitwise operators](https://www.w3schools.com/js/js_bitwise.asp)