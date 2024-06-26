# Discover streamlined location updates

Move into the future with Core Location! Meet the CLLocationUpdate class, designed for modern Swift concurrency, and learn how it simplifies getting location updates. We’ll show you how this class works with your apps when they run in the foreground or background and share some best practices.

@Metadata {
   @TitleHeading("WWDC23")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc23/10180", purpose: link, label: "Watch Video (15 min)")

   @Contributors {
      @GitHubUser(multitudes)
   }
}



## Intro

There is a new Swift native API, with out of box support for modern swift concurrency to get location updates:

```swift
for try await update in CLLocationUpdate.liveUpdates() {
    print("My current location : \(update.location)")
}
```

- CLLocationUpdate
- Getting updates
- Updates in the background
- Automatic pause / resume
- Process lifecycle

# CLLocationUpdate

There is a new class CLLocationUpdate, which has a static function liveUpdates, that returns an AsyncSequence called Updates. Which can directly be iterated using for/try/await, to yield an element of type CLLocationUpdate, that contains a location of type CLLocation, and a boolean flag isStationary for managing automatic, pause and resume. liveUpdates also consumes an optional argument of LiveConfiguration enum type.

![location updates][location1]

[location1]: WWDC23-10180-location1

# Getting updates
The Apple engineer will do some code walkthrough building a basic app that starts location updates from the Foreground. 

- First we import CoreLocation.  
- Then call the static factory function liveUpdates provided by the CLLocationUpdate class to get the Updates AsyncSequence. Which can directly be iterated using for/try/await to get a CLLocationUpdate in the closure.  
- And then get the location by accessing its location property.

At this point our updates have started. How to stop it? 
We break out of the for loop when this isStationary is reported to be true, automatically stopping the updates.

```swift
// Getting Updates
import CoreLocation

let updates = CLLocationUpdate.liveUpdates()

for try await update in updates {
	print ("my location is \(update.location)")
	
	// To stop updates break out of the for loop
	if update.isStationary {
		break
	}
}
```

CLLocationUpdate API is returning a AsyncSequence.

All the powerful things which an AsyncSequence can do like finding, selecting, and excluding elements can also be performed on this updates sequence.

Here, under the hood updates are started and the location of each element is checked for speed. As soon as an update whose speed is more than 200 is found, that first element will be returned, completing the operation. Updates will automatically stop once the first match is found.
```swift
// Getting Updates
import CoreLocation

let updates = CLLocationUpdate.liveUpdates()

let update = try await updates.first(where: {($0.location?.speed ?? 0) > 200.0})

print("speed is : \(update?.location?.speed)")
// Be careful - execution will get stuck until a match is found
```

The speed is too fast in the example? This is 200 meter per second, so roughly around 447 miles per hour. 
Need to be careful while using these filters because execution will get stuck until a match is found...

Back to the sample code from the previous slide. We are getting updates configured automatically with a default config.

```swift
	let updates = CLLocationUpdate.liveUpdates()
```

The liveUpdates API can take an explicit configuration. This configuration is a new enum type.

```swift
	let updates = CLLocationUpdate.liveUpdates(.automotiveNavigation)
```

The members of this enum type: 

```swift
// LiveConfiguration possible values

public enum LiveConfiguration {
	
	case default
	case automotiveNavigation
	case otherNavigation
	case fitness
	case airborne
}
```

If the app is already using a particular CLActivityType with existing location updates API, then we can choose a corresponding LiveConfiguration member to have the same location experience while adopting the new API. But if we don't have the need for a specific activityType, then we can start the updates either with default configuration or without specifying any configuration at all. 
So what does this "updates" AsyncSequence yield? 
Iterating on it, it gives an object, which is of type `CLLocationUpdate`. It contains an optional location of type CLLocation. If no location is available, we get an update with location marked as nil. It also contains a boolean property isStationary through which we can manage automatic pause / resume of location updates.

```swift
public struct CLLocationUpdate : Sendable {
	public let location : CLLocation? // nil if no location is available
	public let isStationary : Bool // true if device becomes stationary
}
```

# Updates in the background

How to get updates when the app is running in the background?  

LiveActivity is the best way to enable background location updates.

![location updates][location2]

[location2]: WWDC23-10180-location2

As long as the LiveActivity remains active, the app can receive updates without any other additional setup. 

No problem if the app doesn't have a LiveActivity yet. Instead we use CLBackgroundActivitySession.

![location updates][location3]

[location3]: WWDC23-10180-location3

when an app authorized as 'While Using' gets updates in the background we get this blue background location indicator. 

![location updates][location4]

[location4]: WWDC23-10180-location4

CLBackgroundActivitySession uses the same indicator to provide background location capability to the app. 

It does so by maintaining the visibility for user regarding location services being used in the background. And since the visibility is maintained, it keeps the app effectively in-use letting it access locations even from the background. CLBackgroundActivitySession supports the app's authorization as a whole. So it will enable the app not only to receive updates while in the background, but also to monitor for events using CLMonitor. 

BackgroundActivitySession has no dependency on Updates being started. Just creating the session displays the indicator when the app is in the background, letting it receive updates and events as needed. 

- Uses background location indicator
- Maintains app visibility
- Keeps app in-use
- Works with:
	- CLLocationUpdate
	- CLMonitor
- CLocationUpdate doesn't need to be started

In order to use CLBackgroundActivitySession, we need to instantiate it and hold it because object deallocation will automatically invalidate the session potentially ending the app's access to background location. The app still needs to have location in its UIBackgroundModes array, in order for BackgroundActivitySession to work effectively. If we don't have an outstanding session, then we must start the new session from the foreground, but we can only rejoin an existing one from the the background. 

Quick code walk through to see how to use backgroundActivitySession.

Before starting the updates, we should instantiate a CLBackgroundActivitySession object to start a new session. Note, we are assigning the session to self.backgroundActivity, which is a property and not to a local variable. This is important because if we used a local variable, then when it goes out of scope, the object it holds would be deallocated, invalidating the session and potentially ending the app's access to location.

```swift
// CLBackgroundActivitySession in action
import CoreLocation

self.backgroundActivity = CLBackgroundActivitySession()

let updates = CLLocationUpdate.liveUpdates()

for try await update in updates {
	...
}
// local variable can go out of scope, invalidating the session

// to end the session
self.backgroudActivity.invalidate ()
```

Then when we want to end our session, we can do that by sending the invalidate message or by letting the object be destroyed. So that's how the app can get updates in the background, either through LiveActivity or through CLBackgroundActivitySession. 

# Automatic pause / resume

How this new API is contributing to battery life by automatically pausing and resuming the updates.

Normally the app is receiving updates while the user is moving, multiple times in a day. But if the device is become stationary, for example, when a user leave their device on the desk, in such a situation the app would get the same locations over and over! We can be power efficient here by pausing the updates. This off loads the app as well, by not giving it redundant locations to process. So once the device is in stationary state for a sufficient amount of time, CLLocationUpdate API will recognize this and trigger Automatic Pause. When pause is triggered, an update will be sent with a non-nil location and isStationary flag marked as True.

![location updates][location5]

[location5]: WWDC23-10180-location5

Later, when the device becomes non-stationary, updates will automatically resume without any user interaction. With this resuming update, we will send isStationary marked as False, continuing the delivery of updates.

![location updates][location6]

[location6]: WWDC23-10180-location6

# Process lifecycle
So automatically pausing and resuming the updates while the app is in background does have an impact on its lifecycle. 

The app while running in foreground and receiving updates may transition from foreground running to background running and vice versa. But now with this new API, at times the app might transition from the background running state to the suspended state. 

This will likely happen when there are no updates to deliver. For example, due to automatic pause, because of a stationary device, or because Location Services is not able to compute a location fix. 

CLLocationUpdate is not going to leave the app in the suspended state. Instead, as soon as updates are available, either because automatic resume kicked in or maybe location is now available, it will unsuspend the app, transitioning it back into the background running state. When the app is resumed from the suspended state, no action is required to continue the updates in background.

![location updates][location7]

[location7]: WWDC23-10180-location7

Suspended is not the only state. It's possible that the app can transition all the way into the terminated state. 

And this transition can happen in several ways. 

- First, directly from background running due to app crash, user close or system termination when resources are constrained. 
- Second, the app can transition into Terminated state even from the suspended state, due to user close or resource constraint.

![location updates][location8]

[location8]: WWDC23-10180-location8

The API can recover the app in most cases, even when it's terminated and not running at all. 
The app will recover as soon as location updates are available and will be relaunched in the background, and this will transition the app from terminated to background running 

But after receiving the launch, there are some steps to perform to make sure the background location session can continue.

Restart the updates by calling `CLLocationUpdate.liveUpdates()` and if the app was previously using a background activity session then also need to recreate a `CLBackgroundActivitySession`. 

It is only possible to rejoin an existing CLBackgroundActivitySession from the background, but not start a new one. 

The just recreated background activity session object it is not the start of a new session. Just a new session object. Now, since the  app has already started a session before it was terminated, this recreation allowed the app to rejoin that existing session from the background, allowing the app to continue the background location updates.

The app should perform any recreation immediately upon receiving background app launch. Needn to place the recreation of these objects somewhere that will get executed when the app receives a background launch. 

For the sample app, Apple placed the recreation in UIApplicationDelegate's - `didFinishLaunchingWithOptions`, which gets called once the app launch has finished. 

There also a sample app to download from the Resources section.

## Resources

[Adopting live updates in Core Location](https://developer.apple.com/documentation/corelocation/adopting_live_updates_in_core_location)  
[Core Location](https://developer.apple.com/documentation/corelocation)  
[Have a question? Ask with tag wwdc2023-10180](https://developer.apple.com/forums/create/question?&tag1=76&tag2=687030)  
[Monitoring location changes with Core Location](https://developer.apple.com/documentation/corelocation/monitoring_location_changes_with_core_location)  
[Search the forums for tag wwdc2023-10180](https://developer.apple.com/forums/tags/wwdc2023-10180)  

## Related Videos 
[Meet Core Location for spatial computing - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10146)  
[Meet Core Location Monitor - WWDC23](https://developer.apple.com/videos/play/wwdc2023/10147)  


