# RxAppState

[![CI Status](http://img.shields.io/travis/pixeldock/RxAppState.svg?style=flat)](https://travis-ci.org/pixeldock/RxAppState)
[![Version](https://img.shields.io/cocoapods/v/RxAppState.svg?style=flat)](http://cocoapods.org/pods/RxAppState)
[![License](https://img.shields.io/cocoapods/l/RxAppState.svg?style=flat)](http://cocoapods.org/pods/RxAppState)
[![Platform](https://img.shields.io/cocoapods/p/RxAppState.svg?style=flat)](http://cocoapods.org/pods/RxAppState)
[![Twitter](https://img.shields.io/badge/Twitter-@pixeldock-blue.svg?style=flat)](http://twitter.com/pixeldock)


A collection of handy RxSwift Observables that let you observe all the changes in your app's state.

## About
In almost every app there is some code that you want to run each time a user opens the app. For example you want to refresh some data or track that the user opened your app.

**UIApplicationDelegate** offers two methods that you could use to run the code when the user opens the app: _applicationWillEnterForeground_ and _applicationDidBecomeActive_. But either of these methods is not ideal for this case:

_applicationWillEnterForeground_ is not called the first time your app is launched. It is only called when the app was in the background and then enters the foreground. At first launch the app is not in the background state so this methods does not get called.

_applicationDidBecomeActive_ does get called when the app is launched for the first time but is also called when the app becomes active after being in inactive state. That happens everytime the user opens Control Center, Notification Center, receives a phone call or a system prompt is shown (e.g. to ask the user for permission to send remote notifications). So if you put your code in _applicationDidBecomeActive_ it will not only get called when the user opens the app but also in all those cases mentioned above.

So to really run your code only when your user opens the app you need to keep track of the app's state. You would probably implement something like this:

```
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    var didEnterBackground = true
    ...
    func applicationDidEnterBackground(_ application: UIApplication) {
        didEnterBackground = true
    }
    func applicationDidBecomeActive(_ application: UIApplication) {
        if didEnterBackground {
            // run your code
            didEnterBackground = false
        }
    }
    ...
}
```
This is not a big problem, but it is not a very elegant approach. And you have to set the inital value of _didEnterBackground_ to _true_ to run your code after the first launch (see above), even if the app never has been to the background. Call me picky, but I don't like that.

**RxAppState to the rescue!**  
With RxAppState you can simply do the following:

```
UIApplication.shared.rx.didOpenApp
    .subscribe(onNext: { _ in
        // run your code
    })
    .addDisposableTo(disposeBag)
```
This runs your code whenever the user opens the app. It includes the first launch of the app and ignores the cases when the app enters active state without having been in background state before (like when the user just opened Control Center or received a phone call)

**And there is more!**  
You want to show your user a tutorial when he first launches the app? And you only want to show it after the first launch and never again? No problem:

```
UIApplication.shared.rx.firstLaunchOnly
    .subscribe(onNext: { _ in
        // run your code
    })
    .addDisposableTo(disposeBag)
```
You want to show your user a message when he opens the app for the first time after an update?

```
UIApplication.shared.rx.firstLaunchOfNewVersionOnly
    .subscribe(onNext: { _ in
        // run your code
    })
    .addDisposableTo(disposeBag)
```

You want to keep track of how many times the user has opened your app? Simply do this:

```
UIApplication.shared.rx.didOpenAppCount
    .subscribe(onNext: { count in
        print("app opened \(count) times")
    })
    .addDisposableTo(disposeBag)
```

**The cherry on top:**   
This code does not have to live in your AppDelegate. You could put it anywhere you like in your app! So don't clutter your AppDelegate with this code, put it somewhere else!

If you are using _firstLaunchOnly_, _isFirstLaunch_ or _didOpenAppCount_ make sure you only subscribe once to each of those 3 Observables. Those Observables store data in NSUserDefault, so if you have more than one subscription you will get incorrect values. 

## Example
There is a simple example project to demonstrate how to use RxAppDelegate.

## Requirements
iOS 8 or greater    
Swift 3.x

## Dependencies
RxSwift 3.0  
RxCocoa 3.0

## Installation
RxAppState is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```ruby
pod "RxAppState"
```

## Author

Jörn Schoppe,  
joern@pixeldock.com   
[![Twitter](https://img.shields.io/badge/Twitter-@pixeldock-blue.svg?style=flat)](http://twitter.com/pixeldock)

## License

RxAppState is available under the MIT license. See the LICENSE file for more info.
