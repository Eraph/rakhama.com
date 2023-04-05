---
title: "Blazor With MVVM"
description: "A guide to setting up Blazor for MVVM"
date: 2023-04-05T14:34:41+10:00
draft: false
categories:
  - Programming
tags:
  - blazor
  - future-reference
  - programming
  - .net
  - tips
series:
  - Professional Development
---
MVVM is a development pattern that has been around for a while now. It was designed to facilitate the development of WPF applications for Windows and is still used for the likes of [Maui](https://learn.microsoft.com/en-us/dotnet/architecture/maui/mvvm) apps. I reckon it has a place in Blazor apps as a neat way to separate view from logic, and as a sort of volatile state management for when you don't want state to persist between pages.

<!--more-->

## Goals
I won't go into detail of what MVVM is or what it can offer, that's [already been explained](https://learn.microsoft.com/en-us/dotnet/architecture/maui/mvvm) better than I can elsewhere. This article also assumes you know what you're doing with your IDE and .NET in general, it also helps to have some familiarity with Blazor.

In summary, MVVM is short for Model, View, ViewModel. From a Blazor application perspective:
- The **Model** would be a DTO, either stored in persistent state management or fetched fresh from a back-end API
- The **View** is the `.razor` file
- The **ViewModel** handles the state and logic of a view, be that a component or a page

Some key benefits of MVVM with regards to a Blazor application:
- Negates (or minimises) the need for code-behind on components and pages
- Complete separation of views from logic; VMs are view agnostic and could, for example, be ported to a Maui app quite easily
- Light state management, e.g. for submission forms that are destroyed when submitted or navigating away
- Use of `INotifyPropertyChanged` and `INotifyCollectionChanged` interfaces streamline page refreshes

For this example I will be using the default Blazor WASM template provided with .NET 6.0. Later versions of .NET should be able to follow the same principles outlined here. I will also be using the following packages:
- [MvvmBlazor](https://www.nuget.org/packages/MvvmBlazor) to easily set up Components and ViewModels
- [CommunityToolkit.Mvvm](https://www.nuget.org/packages/CommunityToolkit.Mvvm) to provide some useful MVVM functionality
- [MudBlazor](https://www.nuget.org/packages/MudBlazor), a component framework with good MVVM integration.

This demonstration won't do anything particularly fancy but should serve as a good introduction to how MVVM can work effectively in a Blazor app.

## Getting Started
Start by creating a Blazor WASM project:
``` bash
dotnet new blazorwasm -n BlazorWithMvvm
```

Go into the new directory and add the three packages mentioned previously:
``` bash
dotnet add package MvvmBlazor
dotnet add package CommunityToolkit.Mvvm
dotnet add package MudBlazor
```

MvvmBlazor and MudBlazor both require additional setup.

### Setting up MvvmBlazor
MvvmBlazor's setup documentation can be found [here](https://github.com/klemmchr/MvvmBlazor).

In `Program.cs`, register the MvvmBlazor service:
``` c#
builder.Services.AddMvvm();
```

Add the relevant MvvmBlazor namespaces to `_Imports.razor`:
``` html
@using MvvmBlazor
@using MvvmBlazor.Components
```

### Setting up MudBlazor
MudBlazor's setup documentation can be found [here](https://www.mudblazor.com/getting-started/installation).

In `Program.cs`, register the MudBlazor service:
``` c#
builder.Services.AddMudServices();
```

Add the MudBlazor namespace to `_Imports.razor`:
``` html
@using MudBlazor
```

Add font and stylesheet references inside the `<head>` tag in `index.html`:
``` html
<link href="https://fonts.googleapis.com/css?family=Roboto:300,400,500,700&display=swap" rel="stylesheet" />
<link href="_content/MudBlazor/MudBlazor.min.css" rel="stylesheet" />
```

Add a script reference alongside the existing Blazor script reference near the bottom of `<body>` in `index.html`:
``` html
<script src="_content/MudBlazor/MudBlazor.min.js"></script>
```

Add the components you need to `MainLayout.razor`; only `MudThemeProvider` is required. This can go underneath the `@inherits...` line.
``` html
<MudThemeProvider/>
<MudDialogProvider/>
<MudSnackbarProvider/>
```

## Creating an MVVM Component

### Create the ViewModel
MvvmBlazor relies on code generators to do a lot of the heavy lifting so VMs are pretty easy to put together. Here's an example (`using` statements and namespace removed for brevity):
``` c#
public partial class MyButtonViewModel : ViewModelBase
{
    [Notify]
    private string _myMessage = default!;

    [Parameter]
    public int CurrentCount { get; set; }

    public ICommand ButtonClick => new AsyncRelayCommand(SetCount, () => CurrentCount > 3);

    private Task SetCount()
    {
        MyMessage = "S'cool!";
        return Task.CompletedTask;
    }
}
```
Here's what you need to know about this ViewModel:
- Source generators require this class to have a namespace
- It must be declared as a `partial` for source generators to work
- It must inherit from `ViewModelBase`
- The `[Notify]` attribute is applied to a member variable, source generators will handle the `INotify` goodies and expose it as a publicly accessible property
- The `[Parameter]` property will tie up with the component itself to pass parameters straight through to the ViewModel
- `ButtonClick` is an `ICommand` which MudBlazor can use to plug into the `Command` property, `AsyncRelayCommand()` is a type provided by CommunityToolkit.Mvvm and in this case takes an action for the first parameter and a predicate for the second parameter; the action will not fire until the predicate is satisfied
- SetCount simply sets the source generated property, triggering notification updates automatically

Add the ViewModel to the services registration in `Program.cs`. In this case I'm using `AddTransient` so that it is disposed of when the component is destroyed; navigating away and back again will reset it.
``` c#
builder.Services.AddTransient<MyButtonViewModel>();
```

{{< alert >}}
You could set this up so that ViewModels can be picked up automatically by using reflection to find all classes with the suffix `ViewModel`, for example.
{{< /alert >}}

### Create the Component
The following code demonstrates how to take advantage of the ViewModel for a component:
``` html
@inherits MvvmComponentBase<MyButtonViewModel>

<MudButton Command="@BindingContext.ButtonClick" Disabled="@(!BindingContext.ButtonClick.CanExecute(null))">Is it cool?</MudButton>

<MudText>
    @Bind(x => x.MyMessage)
</MudText>

@code {
    [Parameter]
    public int CurrentCount { get; set; }
}
```

Here's what you need to know about this Component:
- It inherits from `MvvmComponentBase<ComponentViewModel>`
- The concrete VM can be accessed with the `BindingContext` property
- In the case of `MudButton`, the `Command` property is not sufficient to disable it when `CanExecute()` is `false` so I've set it up manually here
- Properties are bound with the pattern `@Bind(x => x.BindingPropertyName)`
- Code-behind is unavoidable but minimal, other components still need to know what parameters this component can accept, hence the `@code` block

### Using the Component
To demonstrate how this component works I plugged it in to the `Counter.razor` page by adding the following just above the `@code` block:
``` html
<MyButton CurrentCount="currentCount" />
```

`currentCount` is a property on that component already and we're passing it down. The idea is that the user can increment that count, but we'll only see our button enable when the count is higher than 3 (see the predicate in the ViewModel).

### Try it out!
Run the app, go to the `Counter` page and the button should be disabled. Increment the counter a few times and the button should be enabled, clicking it should show a wee message.