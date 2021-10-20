---
title: Install tools for Windows app development
description: This article provides instructions for setting up your development computer for Windows app development.
ms.topic: article
ms.date: 10/05/2021
keywords: windows win32, windows app development, Windows App SDK 
ms.author: zafaraj
author: zaryaf
ms.localizationpriority: medium
---

# Install tools for Windows app development

To develop apps for Windows 11 and Windows 10, you'll need to configure your development computer with the required development tools.

## 1. Check system requirements

To develop apps, you'll need Visual Studio, the Windows SDK, and the Windows App SDK. For a list of the minimum system requirements for each of these tools, see [System requirements for Windows app development](system-requirements.md).

## 2. Install Visual Studio

Use the following links to install Visual Studio 2019 or Visual Studio 2022 (Preview). You can choose between the free Visual Studio Community edition, Visual Studio Professional, or Visual Studio Enterprise. Whichever version you choose, the latest Windows SDK will also be installed by default.

> [!div class="button"]
> [Download Visual Studio 2019](/visualstudio/releases/2019/release-notes)

> [!div class="button"]
> [Download Visual Studio 2022 (Preview)](/visualstudio/releases/2022/release-notes-preview)

### Required workloads and components

Make sure these workloads and components are installed with Visual Studio. These are all selected by default.

- On the **Workloads** tab of the installation dialog, these workloads are required:
  - **Universal Windows Platform development**
  - **Desktop development with C++**
  - **.NET Desktop Development**

- On the **Individual components** tab of the installation dialog, **Windows 10 SDK (10.0.19041.0)** is required in the **SDKs, libraries, and frameworks** section.

- In the **Installation details** pane of the installation dialog, make sure the following items are selected in the **Universal Windows Platform development** section:
  - For Visual Studio 2019: **C++ (v142) Universal Windows Platform tools**
  - For Visual Studio 2022 (Preview): **C++ (v143) Universal Windows Platform tools**

## 3. Enable NuGet Package source

Make sure your system has a NuGet package source enabled for the official NuGet service index at `https://api.nuget.org/v3/index.json`.

 1. In Visual Studio, select **Tools** -> **NuGet Package Manager** -> **Package Manager Settings** to open the **Options** dialog.
 2. In the left pane of the **Options** dialog, select the **Package Sources** tab, and make sure there is a package source for **nuget.org** that points to `https://api.nuget.org/v3/index.json` as the source URL. For more information, see [Common NuGet configurations](/nuget/consume-packages/configuring-nuget-behavior).

## 4. Install the Windows App SDK extension for Visual Studio

The [Windows App SDK](index.md) provides a unified set of APIs, project templates, and other tools for building Windows apps. This SDK is available as a Visual Studio extension (VSIX). You can choose from three versions of the extension: **stable**, **preview**, and **experimental**. For more information about the differences between these versions, see [Release channels](release-channels.md).

> [!NOTE]
> If you have already installed any [previous version](downloads.md) of the Windows App SDK extension for Visual Studio, uninstall the previous extension before installing a new version. For more information about how to uninstall an extension, see [Manage extensions for Visual Studio](/visualstudio/ide/finding-and-using-visual-studio-extensions).

To download the latest Windows App SDK extensions for Visual Studio, see the installation instructions in following tabs. For older versions, see [Downloads](downloads.md).

### [Stable release](#tab/stable)

To develop desktop (C#/.NET 5 or C++) apps that can be used in production environments, install the latest extension from the **stable** release channel. For more information about this channel and the features available in it, see [Stable release channel](stable-channel.md).

Choose one of these options to install the latest stable release ([version 0.8.2](stable-channel.md#version-08)):

- In Visual Studio, click **Extensions** > **Manage Extensions**, search for **Project Reunion**, and install the latest extension.
    > [!NOTE]
    > In the latest available stable release, the Windows App SDK extension for Visual Studio is named **Project Reunion**. In later releases, it has been renamed to **Windows App SDK**.

- Alternatively, you can download and install the extension directly from Visual Studio Marketplace.

    > [!div class="button"]
    > [Download latest stable release](https://aka.ms/windowsappsdk/stable-vsix)

### [Preview release](#tab/preview)

> [!WARNING]
> Versions 1.0 Preview 1 and 2 of the Windows App SDK contain a critical bug that corrupts your system’s PATH variable. We are fixing this in the upcoming 1.0 Preview 3 release. If you’ve already installed one of these previews, see this GitHub thread for [how to resolve the issue](https://github.com/microsoft/WindowsAppSDK/issues/1599). Until Preview 3 is available, we recommend using version [1.0 Experimental](https://aka.ms/windowsappsdk/experimental-vsix) or being aware of this issue and how it might impact your work. 

To install a preview of the next stable release that can be used to develop desktop (C#/.NET 5 or C++) apps, install the latest extensions from the **preview** release channel. For more information about this channel and the features available in it, see [Preview release channel](stable-channel.md). This release channel cannot be used by apps in production environments.

Choose one or more of the following extensions for the latest preview release ([version 1.0 Preview 2](preview-channel.md#version-10-preview-2-100-preview2)), based on the version of Visual Studio you have installed and the programming language you want to use. 

- **Visual Studio 2019**:

   > [!div class="button"]
    > [Download C++ extension](https://aka.ms/windowsappsdk/1.0-preview2/extension/VS2019/cpp)

    > [!div class="button"]
    > [Download C# extension](https://aka.ms/windowsappsdk/1.0-preview2/extension/VS2019/csharp)

- **Visual Studio 2022**:

    > [!div class="button"]
    > [Download C++ extension](https://aka.ms/windowsappsdk/1.0-preview2/extension/VS2022/cpp)

    > [!div class="button"]
    > [Download C# extension](https://aka.ms/windowsappsdk/1.0-preview2/extension/VS2022/csharp)

The extensions from the preview channel are available only from the download locations provided above. These extensions are not available via Visual Studio Marketplace or the **Manage Extensions** dialog box in Visual Studio.

> [!NOTE]
> If you install the C# version of the Windows App SDK 1.0 Preview 2 extension for Visual Studio 2019 and want to use the [single-project MSIX project template](single-project-msix.md), you must also install the [Single-project MSIX Packaging Tools extension](https://marketplace.visualstudio.com/items?itemName=ProjectReunion.MicrosoftSingleProjectMSIXPackagingTools) separately. The **Blank App, Packaged (WinUI 3 in Desktop)** project template has a known issue that results in a build error unless you also install the single-project packaging tools extension. This issue does not affect other versions of the Windows App SDK 1.0 Preview 2 extension.

### [Experimental release](#tab/experimental)

To develop desktop (C#/.NET 5 or C++) apps or UWP apps that use the latest experimental features, install the latest extension from the **experimental** release channel. For more information about this channel and the features available in it, see [Experimental release channel](experimental-channel.md). This release channel cannot be used by apps in production environments.

Choose one of these options to install the latest experimental release ([version 1.0 Experimental](experimental-channel.md#version-10-experimental-100-experimental1)):

- In Visual Studio, click **Extensions** > **Manage Extensions**, search for **Windows App SDK (Experimental)**, and install the latest extension.
- Alternatively, you can download and install the extension directly from Visual Studio Marketplace.

    > [!div class="button"]
    > [Download latest experimental release](https://aka.ms/windowsappsdk/experimental-vsix)

---

## 5. Download Windows App SDK installer and MSIX packages

Unpackaged apps can deploy the Windows App SDK package dependencies by using the Windows App SDK .exe installer or by deploying the MSIX packages directly from the app's setup program. For instructions, see the [deployment guide for unpackaged apps](deploy-unpackaged-apps.md). 

> [!div class="button"]
> [Download latest installer & MSIX packages](https://aka.ms/windowsappsdk/1.0-preview2/msix-installer)

For older versions, see [Downloads](downloads.md).

## 6. Enable your device for development

Before you can deploy apps to your development computer, you have to enable it for development. For detailed instructions, see [Enable your device for development](../get-started/enable-your-device-for-development.md).

## 7. Register as an app developer

You can start developing apps now, but you need a developer account to submit your apps to the Microsoft Store. For more information, see [Create a developer account](../get-started/sign-up.md).

## Other tools and downloads

- To enhance the developer experience for MSIX-packaged desktop applications, you can optionally install the single-project MSIX packaging tools extension for Visual Studio. This extension enables you to develop and build your MSIX-packaged desktop application without requiring a separate packaging project. This extension is installed by default with the Windows App SDK 1.0 Preview 2 and later versions, or it can be installed separately for previous versions of the Windows App SDK. For more information, see [Package your app using single-project MSIX](single-project-msix.md).
- If you want to customize your device and install other features or packages, check out the [developer setup scripts](https://github.com/Microsoft/windows-dev-box-setup-scripts).
- For more tools and downloads, see [Downloads and tools for Windows development](https://developer.microsoft.com/windows/downloads).

## Related topics

- [System requirements for Windows app development](system-requirements.md)
- [Get started developing apps for Windows desktop](../get-started/index.md)
- [Visual Studio project and item templates for Windows apps](../desktop/visual-studio-templates.md)
- [Create a new project that uses the Windows App SDK](../winui/winui3/create-your-first-winui3-app.md)
- [Use the Windows App SDK in an existing project](use-windows-app-sdk-in-existing-project.md)
- [Enable your device for development](../get-started/enable-your-device-for-development.md)
