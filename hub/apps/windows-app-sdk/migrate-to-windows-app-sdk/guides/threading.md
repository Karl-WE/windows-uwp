---
title: Threading functionality migration
description: This topic shows how to migrate your threading code.
ms.topic: article
ms.date: 09/20/2021
keywords: Windows, App, SDK, migrate, migrating, migration, port, porting, windowing
ms.author: stwhi
author: stevewhims
ms.localizationpriority: medium
dev_langs:
  - csharp
  - cppwinrt
---

# Threading functionality migration

This topic shows how to migrate your threading code.

## Change Windows.UI.Core.CoreDispatcher to Microsoft.UI.Dispatching.DispatcherQueue

This section applies if you're using the [**Windows.UI.Core.CoreDispatcher**](/uwp/api/Windows.UI.Core.CoreDispatcher) class in your UWP app. That includes the use of any methods or properties that take or return a **CoreDispatcher**, such as the [**DependencyObject.Dispatcher**](/uwp/api/windows.ui.xaml.dependencyobject.dispatcher) and [**CoreWindow.Dispatcher**](/uwp/api/windows.ui.core.corewindow.dispatcher) properties. For example, you'll be calling **DependencyObject.Dispatcher** when you retrieve the **CoreDispatcher** belonging to a [**Windows.UI.Xaml.Controls.Page**](/uwp/api/windows.ui.xaml.controls.page).

```csharp
// MainPage.xaml.cs in a UWP app
if (this.Dispatcher.HasThreadAccess)
{
    ...
}
```

```cppwinrt
// MainPage.xaml.cpp in a UWP app
if (this->Dispatcher().HasThreadAccess())
{
    ...
}
```

Instead, in your Windows App SDK app, you'll need to use the [**Microsoft.UI.Dispatching.DispatcherQueue**](/windows/winui/api/microsoft.ui.dispatching.dispatcherqueue) class. And the corresponding methods or properties that take or return a **DispatcherQueue**, such as the [**DependencyObject.DispatcherQueue**](/windows/winui/api/microsoft.ui.xaml.dependencyobject.dispatcherqueue) and [**Microsoft.UI.Xaml.Window.DispatcherQueue**](/windows/winui/api/microsoft.ui.xaml.window.dispatcherqueue) properties. For example, you'll be calling **DependencyObject.DispatcherQueue** when you retrieve the **DispatcherQueue** belonging to a [**Microsoft.UI.Xaml.Controls.Page**](/windows/winui/api/microsoft.ui.xaml.controls.page) (most XAML objects are **DependencyObject**s).

```csharp
// MainPage.xaml.cs in a Windows App SDK app
if (this.DispatcherQueue.HasThreadAccess)
{
    ...
}
```

```cppwinrt
// MainPage.xaml.cpp in a Windows App SDK app
#include <winrt/Microsoft.UI.Dispatching.h>
...
if (this->DispatcherQueue().HasThreadAccess())
{
    ...
}
```

## Change CoreDispatcher.RunAsync to DispatcherQueue.TryEnqueue

This section applies if you're using the [**Windows.UI.Core.CoreDispatcher.RunAsync**](/uwp/api/windows.ui.core.coredispatcher.runasync) method to schedule a task to run on the main UI thread (or on the thread associated with a particular [**Windows.UI.Core.CoreDispatcher**](/uwp/api/Windows.UI.Core.CoreDispatcher)).

```csharp
// MainPage.xaml.cs in a UWP app
public void NotifyUser(string strMessage)
{
    if (this.Dispatcher.HasThreadAccess)
    {
        StatusBlock.Text = strMessage;
    }
    else
    {
        var task = this.Dispatcher.RunAsync(
            Windows.UI.Core.CoreDispatcherPriority.Normal,
            () => StatusBlock.Text = strMessage);
    }
}
```

```cppwinrt
// MainPage.cpp in a UWP app
void MainPage::NotifyUser(std::wstring strMessage)
{
    if (this->Dispatcher().HasThreadAccess())
    {
        StatusBlock().Text(strMessage);
    }
    else
    {
        auto task = this->Dispatcher().RunAsync(
            Windows::UI::Core::CoreDispatcherPriority::Normal,
            [strMessage, this]()
            {
                StatusBlock().Text(strMessage);
            });
    }
}
```

In your Windows App SDK app, use the [Microsoft.UI.Dispatching.DispatcherQueue.TryEnqueue](/windows/winui/api/microsoft.ui.dispatching.dispatcherqueue.tryenqueue)) method instead. It adds to the [**Microsoft.UI.Dispatching.DispatcherQueue**](/windows/winui/api/microsoft.ui.dispatching.dispatcherqueue) a task that will be executed on the thread associated with the **DispatcherQueue**.

```csharp
// MainPage.xaml.cs in a Windows App SDK app
public void NotifyUser(string strMessage)
{
    if (this.DispatcherQueue.HasThreadAccess)
    {
        StatusBlock.Text = strMessage;
    }
    else
    {
        var task = this.DispatcherQueue.TryEnqueue(
        Microsoft.UI.Dispatching.DispatcherQueuePriority.Normal,
        () => StatusBlock.Text = strMessage);
    }
}
```

```cppwinrt
// MainPage.xaml.cpp in a Windows App SDK app
#include <winrt/Microsoft.UI.Dispatching.h>
...
void MainPage::NotifyUser(std::wstring strMessage)
{
    if (this->DispatcherQueue().HasThreadAccess())
    {
        StatusBlock().Text(strMessage);
    }
    else
    {
        auto task = this->DispatcherQueue().TryEnqueue(
            Microsoft::UI::Dispatching::DispatcherQueuePriority::Normal,
            [strMessage, this]()
            {
                StatusBlock().Text(strMessage);
            });
    }
}
```

## Migrate winrt::resume_foreground (C++/WinRT)

This section applies if you're using the [**winrt::resume_foreground**](/uwp/cpp-ref-for-winrt/resume-foreground) function (in a coroutine in your C++/WinRT UWP app) to switch execution to the foreground thread associated with a [**Windows.UI.Core.CoreDispatcher**](/uwp/api/Windows.UI.Core.CoreDispatcher).

```
// MainPage.cpp in a UWP app
winrt::fire_and_forget MainPage::ClickHandler(IInspectable const&, RoutedEventArgs const&)
{
    ...
    co_await winrt::resume_foreground(this->Dispatcher());
}
```

In your Windows App SDK app, instead of **CoreDispatcher** you'll need to use the [**Microsoft.UI.Dispatching.DispatcherQueue**](/windows/winui/api/microsoft.ui.dispatching.dispatcherqueue) class, as described in [Change Windows.UI.Core.CoreDispatcher to Microsoft.UI.Dispatching.DispatcherQueue](#change-windowsuicorecoredispatcher-to-microsoftuidispatchingdispatcherqueue).

```cppwinrt
// MainPage.xaml.cpp in a Windows App SDK app
#include <winrt/Microsoft.UI.Dispatching.h> // includes Microsoft.UI.Dispatching.DispatcherQueue
#include <microsoft.ui.dispatching.co_await.h> // includes winrt::resume_foreground support for DispatcherQueue
...
winrt::fire_and_forget MainPage::ClickHandler(IInspectable const&, RoutedEventArgs const&)
{
    ...
    co_await winrt::resume_foreground(this->DispatcherQueue());
}
```
