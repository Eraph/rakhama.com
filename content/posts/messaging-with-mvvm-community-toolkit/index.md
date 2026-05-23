---
title: "Messaging With the MVVM Community Toolkit"
description: "How to effectively use the MVVM Community Toolkit's Messenger feature to decouple your app"
date: 2026-05-23T14:43:39+10:00
draft: false
categories:
  - Programming
tags:
  - future-reference
  - tips
  - programming
  - .net
  - maui
  - mvvm
  - testing
images:
  - "cover.jpg"
---

I've been doing some work with [.NET MAUI](https://learn.microsoft.com/en-us/dotnet/maui/) lately.

I know, I know, it was my idea and I've only got myself to blame. I'll maybe make another post about how not-very-good .NET MAUI is out of the box and how [Avalonia UI](https://avaloniaui.net/) is, in my opinion, what it should have been, but this post is about how to use the poorly documented [MVVM Community Toolkit](https://learn.microsoft.com/en-au/dotnet/communitytoolkit/maui/)'s messaging functionality to decouple my app instead of relying on event handlers.

Long story short, the idea is sound but [the documentation](https://learn.microsoft.com/en-au/dotnet/communitytoolkit/mvvm/) fails to effectively surface some pretty fundamental implementation details. The [sample app](https://aka.ms/mvvmtoolkit/samples) doesn't help much more either.

<!--more-->

## Events, Delegates, and the += Operator

I've adopted the MVVM pattern and I wanted ViewModels to react to changes in data from services, for example when a background API call causes some data to refresh. The traditional way of doing this would look something like the following:

* The service implements the `INotifyPropertyChanged` interface with the delegate `PropertyChangedEventHandler(object? sender, PropertyChangedEventArgs e)`
* The service must trigger the `OnPropertyChanged()` method whenever a property of interest is changed/updated
* The ViewModel has the service injected into the constructor
* The ViewModel must subscribe to the service's `PropertyChanged` event and act accordingly

Even back in the .NET 2.0 days, I wasn't that enamoured with the event subscription pattern. Take a look at this:

``` csharp
_service.PropertyChanged += (sender, event) // (object? sender, PropertyChangedEventArgs event)
{
  Console.WriteLine(sender?.GetType());
  Console.WriteLine(event.PropertyName);
};
```

It's not great! Here are the problems I see:

* One handler for everything; all usage of `OnPropertyChanged()` on the service fires the same event. The handler must distinguish between changed properties and handle them individually (gross)
* No typing; the sender is a nullable object, and nothing in the event argument hints at what type the changed value is
* The changed value is not provided; to get the value, it's up to the subscriber to look up the appropriate property on the service according to the information provided
* Calling `OnPropertyChanged()` from the service is a bit of a manual process and could be easy to forget if the property is updated in multiple places (assuming the call is not part of the property setter)

On top of this, the ViewModel requires the service to be injected so that the subscription can be made. This isn't particularly bad in itself, but if the VM doesn't need the service for any other purpose, it can feel a bit heavy handed.

At least with regards to handling the `PropertyChanged` event firing, the MVVM Community Toolkit does introduce the `[ObservableObject]` and `[ObservableProperty]` attributes. These simplify things a bit from the service end, so instead of handling all the event handler boiler-plate code, you can simply do something like this:

``` csharp
public class MyService : ObservableObject
{
  // This attribute creates a corresponding property called MyChangingProperty
  // The property has logic set up to handle notification events automatically when assigned to
  [ObservableProperty]
  private string _myChangingProperty = string.Empty;

  public void UpdateChangingProperty(string newValue)
  {
    MyChangingProperty = newValue;
  }
}
```

It's better, but the subscription from the ViewModel isn't improved by this. This is where the MVVM Community Toolkit's [Messenger feature](https://learn.microsoft.com/en-us/dotnet/communitytoolkit/mvvm/messenger) comes in.

## Check Your Inbox

The Messenger feature follows a more modern pub/sub pattern, and provides a lot more goodies like dedicated subscriptions per property (assuming distinct types), and rich, strongly-typed notifications that give you not only the correctly typed new value, but the old value as well. It also means that the ViewModels no longer need to know about the services if they're only being used to subscribe to events. You can still inject a service if you want to call an external service from a VM to update data, for example, or you can use the same pattern backwards to send and receive messages, completely decoupling services from ViewModels altogether.

It's easy enough to inject the `IMessenger` implementation into classes that need to send and receive messages, but that's a very manual process of sending and registering wherever it's required. Fortunately, the MVVM Community Toolkit provides much easier ways to handle it for us.

* Classes that inherit from `ObservableRecipient` or are decorated with the `[ObservableRecipient]` attribute are given a `Messenger` property (optionally injected, otherwise it's automatically resolved from `WeakReferenceMessenger.Default`), and other notification helpers.
* Decorating a private field with both `[ObservableProperty]` and `[NotifyPropertyChangedRecipients]` attributes generates a corresponding property that automatically sends messages of type `PropertyChangedMessage<PropertyType>` when the property is assigned to (imagine this in the service, for example when an API call completes)
* The `IRecipient<PropertyChangedMessage<PropertyType>>` interface can be implemented in classes that need to consume the message sent from the aforementioned property change (imagine this in the VM to react when the service has refreshed data)
* Importantly, the receiving class must set its `IsActive` property to true to register the listeners, and it's good practice to unregister the listeners when the object is disposed.

Here's a rough and ready example of how it looks.

### Sending Class

``` csharp
// Must be partial and must either inherit from ObservableRecipient or have the ObservableRecipient attribute
public partial class MessengerSenderService(IMessenger messenger, IApiService apiService) :
  ObservableRecipient(messenger)
{
  // These attributes create a property called MessageData
  [ObservableProperty, NotifyPropertyChangedRecipients]
  private MessageDataClass _messageData = new(); 

  public async Task LoadData()
  {
    // Assigning to MessageData automatically sends a message of type PropertyChangedMessage<MessageDataClass>
    MessageData = await apiService.GetData();
  }
}
```

### Receiving Class

``` csharp
// Must be partial and must either inherit from ObservableRecipient or have the ObservableRecipient attribute
// Despite the names, it is not necessarily a recipient, but those are the features that give the class messaging capability
public partial class MessengerRecipientViewModel :
  ObservableRecipient,
  // For each message type, implement the appropriately typed IRecipient interface
  IRecipient<PropertyChangedMessage<MessageDataClass>>,
  IDisposable
{
  public MessageDataClass ViewModelData = new();

  public MessengerRecipientViewModel(IMessenger messenger) : base(messenger)
  {
    // Critical! Setting IsActive to true registers this class for receiving messages
    IsActive = true;
  }

  // The magic method that handles incoming messages - and it's (mostly) strongly typed, too!
  public void Receive(PropertyChangedMessage<MessageDataClass> message)
  {
    if(message.Sender is MessengerSenderService && message.PropertyName == "MessageData")
    {
      ViewModelData = message.NewValue;
    }
  }

  public void Dispose()
  {
    // It is good practice to unregister messaging from this class when disposing
    IsActive = false;
    GC.SuppressFinalize(this);
  }
}
```

## Test Those Messages!

For unit testing, I'm using XUnit and Moq. Testing when a message is received is easy enough, just call the `Receive()` method and test it does what you expect. But testing sending messages? Well, the MVVM Community Toolkit certainly doesn't make it easy.

Moq can't use the `.Setup()` or `.Verify()` processes on extension methods, which most of the `IMessenger.Send()` implementations are. All those implementations call a core `Send<TMessage, TToken>(TMessage message, TToken token)` method which _should_ be easy enough to set up, until you see that the generic types are bound as `where TMessage : class where TToken : IEquatable<TToken>`.

`TMessage` is easy enough, but look at `TToken` - it is a type of `IEquatable<TToken>`. Well I dug a bit deeper to find the implementation of this, and it's basically [this struct called Unit](https://github.com/CommunityToolkit/dotnet/blob/main/src/CommunityToolkit.Mvvm/Messaging/Internals/Unit.cs). Cool, I'll just use this in... oh, it's marked `internal`. No dice. Alright, I'll just try and fudge `TToken` into a Moq setup/verify statement. No, it doesn't match up with `IEquatable<TToken>`. Okay, I'll use `IEquatable<TToken>`. No, it doesn't match up with `TToken`. Well, this had me banging my head against my desk for quite some time.

The solution, courtesy of [this Stack Overflow answer](https://stackoverflow.com/a/70949014), is to write a custom Moq `TypeMatcher`:

``` csharp
[TypeMatcher]
public sealed class MessengerToken :
  ITypeMatcher,
  IEquatable<MessengerToken>
{
  public bool Matches(Type typeArgument) => true;
  public bool Equals(MessengerToken? other) => throw new NotImplementedException();
}
```

And use it like below:

``` csharp
public class MessengerSenderTests
{
  private readonly MessengerSenderService _messengerSenderService;
  private readonly Mock<IMessenger> _messenger;
  private readonly Mock<IApiService> _apiService;

  public MessengerSenderTests()
  {
    _messenger = new Mock<IMessenger>();
    _apiService = new Mock<IApiService>();

    _messengerSenderService = new MessengerSenderService(_messenger.Object, _apiService.Object);
  }

  [Fact]
  public async Task WhenDataLoaded_MessageIsSent()
  {
    MessageDataClass apiResponse = new MessageDataClass();

    _apiService.Setup(a => a.GetData()).ReturnsAsync(apiResponse);

    await _messengerSenderService.LoadData();

    _messenger.Verify(m => m.Send(It.Is<PropertyChangedMessage<MessageDataClass>>(m =>
        m.NewValue == apiResponse &&
        m.PropertyName == "MessageData" &&
        m.Sender == _messengerSenderService),
      It.IsAny<MessengerToken>()), // The Secret Sauce
      Times.Once());
    _messenger.VerifyNoOtherCalls();
  }
}
```

Thank you, StackOverflow stranger for your wisdom, and Moq for your flexibility.

## It Is Done

Hurray! Now Services and ViewModels may be completely decoupled, subscriptions are strongly typed, and tests can be performed against the sending and receiving parts easily. .NET MAUI and the MVVM Community Toolkit aren't the best put together and the documentation is even worse, so I hope this article was able to demistify some of the features and that you find it helpful!