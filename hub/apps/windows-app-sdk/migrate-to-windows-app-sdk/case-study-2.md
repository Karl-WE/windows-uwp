---
title: A Windows App SDK migration of the UWP Photo Editor sample app (C++/WinRT)
description: A case study of taking the C++/WinRT [UWP Photo Editor sample app](/samples/microsoft/windows-appsample-photo-editor/photo-editor-cwinrt-sample-application/), and migrating it to the Windows App SDK.
ms.topic: article
ms.date: 10/01/2021
keywords: Windows, App, SDK, migrate, migrating, migration, port, porting, C++/WinRT, Photo, Editor, UWP
ms.author: stwhi
author: stevewhims
ms.localizationpriority: medium
---

# A Windows App SDK migration of the UWP Photo Editor sample app (C++/WinRT)

This topic is a case study of taking the C++/WinRT [UWP Photo Editor sample app](/samples/microsoft/windows-appsample-photo-editor/photo-editor-cwinrt-sample-application/), and migrating it to the Windows App SDK.

Begin by [cloning the UWP sample app's repo](https://github.com/microsoft/windows-appsample-photo-editor/tree/master/), and opening the solution in [Visual Studio](https://visualstudio.microsoft.com/downloads/).

> [!IMPORTANT]
> For considerations and strategies for approaching the migration process, and how to set up your development environment for migrating, see [Overall migration strategy](overall-migration-strategy.md).

## Install the Windows App SDK VSIX

Download the Windows App SDK Visual Studio extension (VSIX) installer from [Stable release channel for the Windows App SDK](/windows/apps/windows-app-sdk/stable-channel), and run to install it.

## Create a new project

In Visual Studio, create a new C++/WinRT project from the **Blank App, Packaged (WinUI 3 in Desktop)** project template. Name the project *PhotoEditor*, uncheck **Place solution and project in the same directory**, and target the most recent release (not preview) of the client operating system.

> [!NOTE]
> We'll be referring to the UWP version of the sample project (the one that you cloned from its [repo](https://github.com/microsoft/windows-appsample-photo-editor/tree/master/)) as the *source* solution/project. We'll be referring to the Windows App SDK version as the *target* solution/project.

## The order in which we'll migrate the code

**MainPage** is an important and prominent piece of the app. But if we were to begin by migrating that, then we'd soon realize that **MainPage** has a dependency on the **DetailPage** view; and then that **DetailPage** has a dependency on the **Photo** model. So for this walkthrough we'll take this approach.

* We'll begin by copying over the asset files.
* Then we'll migrate the **Photo** model.
* Next we'll migrate the **App** class (since that needs some members adding to it that **DetailPage** and **MainPage** will depend on).
* Then we'll begin migrating the views, starting with **DetailPage** first.
* And we'll finish up by migrating the **MainPage** view.

### We'll be copying entire source code files

In this walkthrough we'll be copying over source code files using **File Explorer**. If you prefer to copy file *contents* over, then see the [Appendix: copying the contents of the **Photo** model's files](#appendix-copying-the-contents-of-the-photo-models-files) section at the end of this topic for an example of how you could do that for **Photo** (and you could then apply a similar procedure to other types in the project). That option does involve a lot more steps, though.

## Copy asset files

In your clone of the source project, in **File Explorer**, locate the folder **Windows-appsample-photo-editor** > **PhotoEditor** > **Assets**. You'll find eight asset files in that folder. Select those eight files, and copy them to the clipboard.

Also in **File Explorer**, now locate the corresponding folder in the target project that you created. The path to that folder is **PhotoEditor** > **PhotoEditor** > **Assets**. Paste into that folder the asset files that you just copied, and accept the prompt to replace the seven files that already exist in the destination.

In your target project in Visual Studio, in **Solution Explorer**, expand the **Assets** folder. Add to that folder the existing `bg1.png` asset file that you just pasted. You can hover the mouse pointer over the asset files. A thumbnail preview will appear for each, confirming that you've replaced/added the asset files correctly.

## Migrate the Photo model

**Photo** is a runtime class that represents a photo. It's a *model* (in the sense of models, views, and viewmodels).

### Copy Photo source code files

In your clone of the source project, in **File Explorer**, locate the folder **Windows-appsample-photo-editor** > **PhotoEditor**. In that folder you'll find the three source code files `Photo.idl`, `Photo.h`, and `Photo.cpp`; those files together implement the **Photo** runtime class. Select those three files, and copy them to the clipboard.

In Visual Studio, right-click the target project node, and click **Open Folder in File Explorer**. This opens the target project folder in **File Explorer**. Paste into that folder the three files that you just copied.

Back in **Solution Explorer**, with the target project node selected, make sure that **Show All Files** is toggled on. Right-click the three files that you just pasted, and click **Include In Project**. Toggle **Show All Files** off.

In the source project, in **Solution Explorer**, `Photo.h` and `.cpp` are nested under `Photo.idl` to indicate that they're generated from (dependent upon) it. If you like that arrangement, then you can do the same thing in the target project by manually editing `\PhotoEditor\PhotoEditor\PhotoEditor\PhotoEditor.vcxproj`. Find the following:

```xml
<ClInclude Include="Photo.h" />
```

And replace it with this:

```xml
<ClInclude Include="Photo.h">
  <DependentUpon>Photo.idl</DependentUpon>
</ClInclude>
```

Repeat that for `Photo.cpp`.

### Migrate Photo source code

In `Photo.idl`, search for the namespace name `Windows.UI.Xaml` (which is the namespace for UWP XAML), and change that to `Microsoft.UI.Xaml` (which is the namespace for WinUI XAML).

> [!NOTE]
> The [Mapping UWP APIs to the Windows App SDK](api-mapping-table.md) topic provides a mapping of UWP APIs to their Windows App SDK equivalents. The change we made above is an example of a namespace name change necessary during the migration process.

In `Photo.cpp`, add `#include "Photo.g.cpp"` to the existing include directives, immediately after `#include "Photo.h"`. This is one of the [Folder and file name differences (C++/WinRT)](overall-migration-strategy.md#folder-and-file-name-differences-cwinrt) to be aware of between UWP and Windows App SDK projects.

Make the following find/replacement (match case and whole word) in the contents of all of the source code in the files that you just copied and pasted.

* `Windows::UI::Xaml` => `Microsoft::UI::Xaml`

From `pch.h` in the source project, copy the following includes, and paste them into `pch.h` in the target project. This is a subset of the header files included in the source project; these are just the headers we need to support the code we've migrated so far.

```cppwinrt
#include <winrt/Microsoft.UI.Xaml.Media.Imaging.h>
#include <winrt/Windows.Storage.h>
#include <winrt/Windows.Storage.FileProperties.h>
#include <winrt/Windows.Storage.Streams.h>
```

Now confirm that you can build the target solution (but don't run yet).

## Migrate the App class

No changes are necessary to `App.idl` and `App.xaml`. But we do need to edit `App.xaml.h` and `.xaml.cpp` to add some new members to the **App** class. We'll do so in a way that lets us build after each section.

### Making the main window object available

In this step we'll make the change that's explained in [Change Windows.UI.Xaml.Window.Current to App.Window](guides/winui3.md#change-windowsuixamlwindowcurrent-to-appwindow).

In the target project, **App** stores the main window object in its private data member *window*. Later in the migration process (when we migrate the source project's use of **Window.Current**), it'll be convenient if that *window* data member is static; and is also made available via an accessor function. So we'll make those changes next. Since we're making *window* static, we'll need to initialize it in `App.xaml.cpp` instead of via the default member initializer that the code is currently using. Here are what those changes look like in `App.xaml.h` and `App.xaml.cpp`.

```cppwinrt
// App.xaml.h
...
struct App : AppT<App>
{
    ...
    static winrt::Microsoft::UI::Xaml::Window Window(){ return window; };

private:
    static winrt::Microsoft::UI::Xaml::Window window;
};
...

// App.xaml.cpp
...
winrt::Microsoft::UI::Xaml::Window App::window{ nullptr };
...
```

### App::OnNavigationFailed

The *Photo Editor* sample app uses navigation logic to navigate between **MainPage** and **DetailPage**. For more info about Windows App SDK apps that need navigation (and those that don't), see [Do I need to implement page navigation?](guides/winui3.md#do-i-need-to-implement-page-navigation).

So the members we'll migrate in the next few sections all exist to support navigation within the app. Let's begin by migrating the **OnNavigationFailed** event handler. Just copy the declaration and the definition of that member function from the source project, and paste it into the target project (in `App.xaml.h` and `App.xaml.cpp`).

In the code you pasted, change `Windows::UI::Xaml` to `Microsoft::UI::Xaml`.

### App::CreateRootFrame

The source project contains a helper function named **CreateRootFrame**. Copy the declaration and the definition of that helper function from the source project, and paste it into the target project (in `App.xaml.h` and `App.xaml.cpp`).

In the code you pasted, change `Windows::UI::Xaml` to `Microsoft::UI::Xaml`. Also change the two occurrences of `Window::Current()` to `window` (which is the name of the data member of the **App** class that we saw earlier).

### App::OnLaunched

The target project already contains an implementation of the **OnLaunched** event handler. Its parameter is a constant reference to a **Microsoft::UI::Xaml::LaunchActivatedEventArgs**, which is correct for the Windows App SDK (contrast that to the source project, which uses **Windows::ApplicationModel::Activation::LaunchActivatedEventArgs**, which is correct for UWP).

We just need to merge the two definitions (source and target) of **OnLaunched** so that **App::OnLaunched** in `App.xaml.cpp` in the target project looks like the listing below. Note that it uses `window` instead of `Window::Current()` (like the UWP version does).

```cppwinrt
void App::OnLaunched(LaunchActivatedEventArgs const&)
{
    window = make<MainWindow>();

    Frame rootFrame = CreateRootFrame();
    if (!rootFrame.Content())
    {
        rootFrame.Navigate(xaml_typename<PhotoEditor::MainPage>());
    }

    window.Activate();
}
```

## Migrate the DetailPage view

**DetailPage** is the class that represents the photo editor page, where Win2D effects are toggled, set, and chained together. You get to the photo editor page by selecting a photo thumbnail on **MainPage**. **DetailPage** is a *view* (in the sense of models, views, and viewmodels).

### Reference the Win2D NuGet package

To support code in **DetailPage**, the source project has a dependency on [Win2D](https://microsoft.github.io/Win2D/WinUI3/html/Introduction.htm). So we'll also need a dependency on Win2D in our target project. But Win2D itself has a dependency on [DWriteCore](/windows/win32/directwrite/dwritecore-overview). So let's add the NuGet packages in the following order.

* In the target solution in Visual Studio, click **Tools** > **NuGet Package Manager** > **Manage NuGet Packages for Solution...** > **Browse**, and type or paste *Microsoft.WindowsAppSDK.DWrite*. Select the correct item in search results, check the *PhotoEditor* project, and click **Install** to install the package into that project.
* Repeat those steps for the *Microsoft.Graphics.Win2D* NuGet package.

### Copy DetailPage source code files

In your clone of the source project, in **File Explorer**, locate the folder **Windows-appsample-photo-editor** > **PhotoEditor**. In that folder you'll find the four source code files `DetailPage.idl`, `DetailPage.xaml`, `DetailPage.h`, and `DetailPage.cpp`; those files together implement the **DetailPage** view. Select those four files, and copy them to the clipboard.

In Visual Studio, right-click the target project node, and click **Open Folder in File Explorer**. This opens the target project folder in **File Explorer**. Paste into that folder the four files that you just copied.

In **File Explorer** change the names of `DetailPage.h` and `DetailPage.cpp` to `DetailPage.xaml.h` and `DetailPage.xaml.cpp`, respectively. This is one of the [Folder and file name differences (C++/WinRT)](overall-migration-strategy.md#folder-and-file-name-differences-cwinrt) to be aware of between UWP and Windows App SDK projects.

Back in **Solution Explorer**, with the target project node selected, make sure that **Show All Files** is toggled on. Right-click the four files that you just pasted (and renamed), and click **Include In Project**. Toggle **Show All Files** off.

In the source project, in **Solution Explorer**, `DetailPage.idl` is nested under `DetailPage.xaml`. If you like that arrangement, then you can do the same thing in the target project by manually editing `\PhotoEditor\PhotoEditor\PhotoEditor\PhotoEditor.vcxproj`. Find the following:

```xml
<Midl Include="DetailPage.idl" />
```

And replace it with this:

```xml
<Midl Include="DetailPage.idl">
  <DependentUpon>DetailPage.xaml</DependentUpon>
</Midl>
```

### Migrate DetailPage source code

In `DetailPage.idl`, search for `Windows.UI.Xaml`, and change that to `Microsoft.UI.Xaml`.

In `DetailPage.xaml.cpp`, change `#include "DetailPage.h"` to `#include "DetailPage.xaml.h"`.

Immediately below that, add `#include "DetailPage.g.cpp"`.

Make the following find/replacements (match case and whole word) in the contents of all of the source code in the files that you just copied and pasted.

* `Windows::UI::Composition` => `Microsoft::UI::Composition`
* `Windows::UI::Xaml` => `Microsoft::UI::Xaml`
* `Window::Current()` => `App::Window()`

For the call to the static **App::Window** method to compile, you'll need to edit `DetailPage.xaml.cpp`, and add `#include "App.xaml.h"` immediately before `#include "Photo.h"`.

From `pch.h` in the source project, copy the following includes, and paste them into `pch.h` in the target project.

```cppwinrt
#include <winrt/Microsoft.Graphics.Canvas.Effects.h>
#include <winrt/Microsoft.UI.Composition.h>
```

Also, at the top of `pch.h`, immediately after `#pragma once`, add this:

```cppwinrt
// This is required because we are using std::min and std::max, otherwise 
// we have a collision with min and max macros being defined elsewhere.
#define NOMINMAX
```

Confirm that you can build the target solution (but don't run yet).

## Migrate the MainPage view

The main page of the app represents the view that you see first when you run the app. It's the page that loads the photos from the **Pictures Library**, and displays a tiled thumbnail view.

### Copy MainPage source code files

Similar to what you did with **DetailPage**, now copy over `MainPage.idl`, `MainPage.xaml`, `MainPage.h`, and `MainPage.cpp`. Rename the `.h` and `.cpp` files to `.xaml.h` and `.xaml.cpp`, respectively. Include all four files in the target project.

In the source project, in **Solution Explorer**, `MainPage.idl` is nested under `MainPage.xaml`. If you like that arrangement, then you can do the same thing in the target project by manually editing `\PhotoEditor\PhotoEditor\PhotoEditor\PhotoEditor.vcxproj`. Find the following:

```xml
<Midl Include="MainPage.idl" />
```

And replace it with:

```xml
<Midl Include="MainPage.idl">
  <DependentUpon>MainPage.xaml</DependentUpon>
</Midl>
```

### Migrate MainPage source code

In `MainPage.idl`, search for `Windows.UI.Xaml`, and change both occurrences to `Microsoft.UI.Xaml`.

In `MainPage.xaml.cpp`, change `#include "MainPage.h"` to `#include "MainPage.xaml.h"`.

Immediately below that, add `#include "MainPage.g.cpp"`.

For the next step, we'll make the change that's explained in [ContentDialog, and Popup](guides/winui3.md#contentdialog-and-popup). So, still in `MainPage.xaml.cpp`, in the **MainPage::GetItemsAsync** method, immediately after the line `ContentDialog unsupportedFilesDialog{};`, add this line of code.

```cppwinrt
unsupportedFilesDialog.XamlRoot(this->Content().XamlRoot());
```

Make the following find/replacements (match case and whole word) in the contents of all of the source code in the files that you just copied and pasted.

* `Windows::UI::Composition` => `Microsoft::UI::Composition`
* `Windows::UI::Xaml` => `Microsoft::UI::Xaml`
* `Window::Current()` => `App::Window()`

For the call to the static **App::Window** method to compile, you'll need to edit `MainPage.xaml.cpp`, and add `#include "App.xaml.h"` immediately before `#include "Photo.h"`.

From `pch.h` in the source project, copy the following includes, and paste them into `pch.h` in the target project.

```cppwinrt
#include <winrt/Microsoft.UI.Xaml.Hosting.h>
#include <winrt/Windows.Storage.Search.h>
```

Confirm that you can build the target solution (but don't run yet).

## Update MainWindow

In `MainWindow.xaml`, delete the **StackPanel** and its contents, since we don't need any UI in **MainWindow**. That leaves only the empty **Window** element.

In `MainWindow.idl`, delete the placeholder `Int32 MyProperty;`, leaving only the constructor.

In `MainWindow.xaml.h` and `MainWindow.xaml.cpp`, delete the declarations and definitions of the placeholder **MyProperty** and **myButton_Click**, leaving only the constructor.

Those are the last of the changes we need to make to migrate the *Photo Editor* sample app. In the next section we'll confirm that we've correctly followed the steps.

## Test the migrated app

Confirm that you can build the target solution. Now build and test the app. Select an image, set a zoom level, choose effects, and configure them.

## Appendix: copying the contents of the **Photo** model's files

As we discussed earlier, you have the option to copy over source code *files* themselves, or the *contents* of source code files. We've already shown how to copy over source code *files* themselves. So this section gives an example of copying file *contents*.

In the source project in Visual Studio, locate the folder **PhotoEditor (Universal Windows)** > **Models**. That folder contains the files `Photo.idl`, `Photo.h`, and `Photo.cpp`, which together implement the **Photo** runtime class.

### Add the IDL, and generate stubs

In your target project in Visual Studio, add a new **Midl File (.idl)** item to the project. Name the new item `Photo.idl`. Delete the default contents of `Photo.idl`.

From the source project in Visual Studio, copy the contents of **Models** > `Photo.idl`, and paste them into the `Photo.idl` file that you just added to your target project. In the code you pasted, search for `Windows.UI.Xaml`, and change that to `Microsoft.UI.Xaml`.

Save the file.

> [!IMPORTANT]
> We're about to perform a build of your target solution. The build won't run to completion at this point, but it will get far enough to do necessary work for us.

Now build the target solution. Even though it won't complete, building is necessary now because it'll generate the source code files (stubs) that we need to get started implementing the **Photo** model.

In Visual Studio, right-click the target project node, and click **Open Folder in File Explorer**. This opens the target project folder in **File Explorer**. There, navigate into the `Generated Files\sources` folder (so you'll be in `\PhotoEditor\PhotoEditor\PhotoEditor\Generated Files\sources`). Copy the stub files `Photo.h` and `.cpp`, and paste them into the project folder, which is now up two folder levels in `\PhotoEditor\PhotoEditor\PhotoEditor`.

Back in **Solution Explorer**, with the target project node selected, make sure that **Show All Files** is toggled on. Right-click the stub files that you just pasted (`Photo.h` and `.cpp`), and click **Include In Project**. Toggle **Show All Files** off.

You'll see a `static_assert` at the top of the contents of `Photo.h` and `.cpp`, which you'll need to delete.

Confirm that you can build again (but don't run yet).

### Migrate code into the stubs

Copy the contents of `Photo.h` and `.cpp` from the source project into the target project.

From here, the remaining steps to migrate the code that you copied are the same as those given in the [Migrate **Photo** source code](#migrate-photo-source-code) section.

## Related topics

* [UWP Photo Editor sample app](/samples/microsoft/windows-appsample-photo-editor/photo-editor-cwinrt-sample-application/)
* [Overall migration strategy](overall-migration-strategy.md)
* [Mapping UWP APIs to the Windows App SDK](api-mapping-table.md)
* [Windows UI Library (WinUI) migration](guides/winui3.md)
