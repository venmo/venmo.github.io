---
layout: post
title: "Thank you, open-source community"
author: Ayaka Nonaka
---

We open-source a lot of things at Venmo, and we also use a lot of open-source libraries. To show some appreciation to all of the open-source maintainers and contributors out there, we thought it might be cool to share a list of open-source libraries and tools that the Venmo iOS team relies on. Thank you everyone out there who has contributed to these projects and other projects out there. Here’s to a great 2015!

* [1PasswordExtension](https://github.com/AgileBits/onepassword-app-extension) adds 1Password support to our login. (Psssst... Look out for it in our next release. ;))
* [AFNetworking](https://github.com/AFNetworking/AFNetworking) is great for all things networking, but we especially like `UIImageView+AFNetworking` for async image loading.
* [Alcatraz](https://github.com/supermarin/Alcatraz) is the package manager that brings XVim to Xcode, and that makes me happy.
* [BZGFormViewController](https://github.com/benzguo/BZGFormViewController) is great for simple forms that require validation. We use it in for in our sign up and edit profile views.
* [BZGMailgunEmailValidation](https://github.com/benzguo/BZGMailgunEmailValidation) is perfect if you use Mailgun for email validation.
* [BlocksKit](https://github.com/zwaldowski/BlocksKit) because `dismissWithClickedButtonIndex:animated:` delegate methods are no fun at all. Besides, who “clicks” on an iOS device?
* [Braintree](https://github.com/braintree/braintree_ios) allows our users to pay via Venmo on apps like [YPlan](https://yplanapp.com/).
* [CMPopTipView](https://github.com/chrismiles/CMPopTipView) has been useful when we want to draw attention to a new feature that we added.
* [CocoaPods](https://github.com/cocoapods/cocoapods) is how we manage all of our dependencies, including private pods. Can’t wait for their 0.36 release which supports Swift!
* [DAZABTest](https://github.com/dasmer/DAZABTest) provides a super simple API to do basic A/B tests.
* [Expecta](https://github.com/specta/expecta) is a great matcher framework that makes your tests read like English. `expect(myTests).toNot.beEmpty()`
* [FLEX](https://github.com/Flipboard/FLEX) is built into all of our debug and dogfood builds, and it makes finding and fixing UI bugs so much fun and so much easier.
* [FXBlurView](https://github.com/nicklockwood/FXBlurView) makes it super easy to have your own blurred views to match iOS 7 and 8’s frosty look.
* [FrameAccessor](https://github.com/AlexDenisov/FrameAccessor) is perfect for the lazy programmer who would prefer to type `view.width = 50` intead of `CGRect newFrame = view.frame; newFrame.size.width = 50; view.frame = newFrame;`
* [GBDeviceInfo](https://github.com/lmirosevic/GBDeviceInfo) tells us useful things about the device our app is running on so we can optimize a little for older devices, etc.
* [Godzippa](https://github.com/mattt/Godzippa) has been immensely helpful when uploading large amounts of data to our API.
* [iRate](https://github.com/nicklockwood/iRate) because we love hearing back from our users. <3
* [JTSImageViewController](https://github.com/jaredsinclair/JTSImageViewController) is what you see when you tap on a picture on a friend’s profile. We love the interaction where you can flick the image off the screen.
* [KGStatusBar](https://github.com/kevingibbon/KGStatusBar) We use this to show “offline mode” in the status bar for Braintree merchants to test out Venmo integration. Super simple to use!
* [KIF](https://github.com/kif-framework/KIF) makes writing automated UI tests such a fun experience. It looks like magic!
* [libextobjc](https://github.com/jspahrsummers/libextobjc) provides us with things like `@weakify` and `@strongify`, and one of our favorites is `EXTKeyPathCoding` which lets us avoid `@"stringlyTyped"`. For example, `[NSSortDescriptor sortDescriptorWithKey:@keypath([Story new], createdDate) ascending:NO]` which gets checked at compile time, as opposed to `[NSSortDescriptor sortDescriptorWithKey:@"createdDate" ascending:NO]` which is prone to typos and harder to refactor.
* [Mantle](https://github.com/Mantle/Mantle) makes converting JSON reponses to and from objects a breeze.
* [MCDateExtensions](https://github.com/mirego/MCDateExtensions) adds some nice additions to NSDate that make it a lot more manageable to do date computations, etc.
* [MMDrawerController](https://github.com/mutualmobile/MMDrawerController) is really easy if you want add drawer navigation to your app. We’re looking to bid farewell to our hamburger button in the near future though.
* [MZFormSheetController](https://github.com/m1entus/MZFormSheetController) brings iPad’s UIModalPresentationFormSheet and brings it to iPhone.
* [Mixpanel](https://github.com/mixpanel/mixpanel-iphone) has a really nice dashboard and handles all of our analytics.
* [Nocilla](https://github.com/luisobo/Nocilla) is our favorite HTTP stubbing libary because it has such a simple and elegant API.
* [NSURL+QueryDictionary](https://github.com/itsthejb/NSURL-QueryDictionary) makes it easy to convert URL query params to a dictionary and vice versa.
* [ObjectiveSugar](https://github.com/supermarin/objectivesugar) is exactly as it sounds. Add some sugar to your Objective-C!
* [OCMock](https://github.com/erikdoe/ocmock) because dependencies. Though with Swift and its focus on value types, we might be using fewer mocks!
* [PSTAlertController](https://github.com/steipete/PSTAlertController) provides an API similar to iOS 8’s UIAlertController, and it’s backwards compatible with iOS 7.
* [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) is a different way of thinking about architecture, and we like it. We’re moving more and more towards FRP.
* [Specta](https://github.com/specta/specta) allows our specs to read like English. `it(@"should allow the user to log in", ^{ ... });` instead of `testUserShouldBeAbleToLogIn`. We think it’s a lot nicer thanReadingABunchOfCamelCasedSentences.
* [SSKeychain](https://github.com/soffes/sskeychain) provides an elegant abstraction around Apple’s Keychain Services API.
* [SVProgressHUD](https://github.com/TransitApp/SVProgressHUD) is basically every spinning progress loader in our app.
* [TOWebViewController](https://github.com/TimOliver/TOWebViewController) is so nice for the few web views that we have in our app, although we’re slowly but surely moving away from them.
* [TTTAttributedLabel](https://github.com/TTTAttributedLabel/TTTAttributedLabel) appears over and over in our app. Any stylized, tappable looking sections of UILabel’s in our app is probably a `TTTAttributedLabel`.
* [Underscore.m](https://github.com/robb/Underscore.m) is one of my personal favorites. Never write a for-loop again.
* [xcpretty](https://github.com/supermarin/xcpretty) makes our Travis CI build output so much cleaner and prettier.
* [XVim](https://github.com/XVimProject/XVim) I think I’m the only one who thinks this on my team, but it’s impossible to navigate quickly around Xcode without this.

A huge thank you to everyone who contributes to open-source. It makes development so much more collaborative, faster, and fun.

## Giving back

Since we use so many open-source libraries, we wanted to give a little back to the community. We hope you find something that you like!

* [DryDock](https://github.com/venmo/DryDock-iOS/) is basically our internal “App Store” for distributing beta builds to the rest of the team.
* [synx](https://github.com/venmo/synx) reorganizes your Xcode project folder to match your Xcode groups, because Xcode doesn’t already do that for some reason.
* [slather](https://github.com/venmo/slather) lets you measure test coverage in your iOS projects and optionally hook it into CI.
* [VENCalculatorInputView](https://github.com/venmo/VENCalculatorInputView) is the calculator keyboard for the amount field of a payment flow.
* [VENPromotionsManager](https://github.com/venmo/VENPromotionsManager) is what we use for location / time dependent events, like our SXSW promotion last year.
* [VENSeparatorView](https://github.com/venmo/VENSeparatorView) is the zig-zaggy line that shows up in your payment feed for cash out events, etc.
* [VENTokenField](https://github.com/venmo/VENTokenField) is the Messages.app style recipients field that we use in our payment and invite flows.
* [VENTouchLock](https://github.com/venmo/VENTouchLock) is our Touch ID + pin code integration.
* [VENVersionTracker](https://github.com/venmo/VENVersionTracker) is what we use to auto-update our dogfood versions. We’re looking to move to using HockeyApp’s equivalent though.
* [Venmo-iOS-SDK](https://github.com/venmo/venmo-ios-sdk) let you build apps that integrate Venmo payments really easily.
