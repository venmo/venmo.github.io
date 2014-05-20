---
layout: post
title: "Blissful UI programming with ReactiveCocoa"
author: Ben Guo
---

We've recently started using [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa) in the Venmo iOS app, and we've found that it provides an expressive, unified alternative to patterns like callback blocks, delegate methods, target-action, notifications, and KVO. If you haven't heard of ReactiveCocoa or Functional Reactive Programming (FRP), we recommend starting with [this awesome post by Josh Abernathy](http://blog.maybeapps.com/post/42894317939/input-and-output) and [the official introduction](https://github.com/ReactiveCocoa/ReactiveCocoa#introduction). In this post, we'll walk through implementing a simple reactive user interface, with and without ReactiveCocoa. Hopefully, we'll inspire you to start experimenting with FRP on your own!

We'll be working on a simple signup form that looks like this:

<img src="/images/reactive_signup.png" width="300px"/>

Let's start with the top of the form, where the user can add or change their profile photo.

{% highlight objc %}
//  SignupViewController.h
// ...
@property (weak, nonatomic) IBOutlet UIImageView *imageView;
@property (weak, nonatomic) IBOutlet UIButton *photoButton;
// ...
{% endhighlight %}

The `photoButton`'s text should be either "Add photo" or "Change photo", depending on whether or not a photo has been added, and the `imageView`'s image should be the currently chosen photo. Here's a reasonable imperative implementation:

##(1a) Imperative

{% highlight objc %}
//  SignupViewController.m

#import "SignupViewController.h"

@interface SignupViewController ()

@property (strong, nonatomic) UIImage *photo;

@end

@implementation SignupViewController

- (void)viewDidLoad {
    [self.photoButton addTarget:self 
                         action:@selector(photoButtonAction) 
               forControlEvents:UIControlEventTouchUpInside];
}

- (void)setPhoto:(UIImage *)photo {
    _photo = photo;
    self.imageView.image = photo;
    if (photo) {
        [self.photoButton setTitle:@"Change photo" 
                          forState:UIControlStateNormal];
    } else {
        [self.photoButton setTitle:@"Add photo" 
                          forState:UIControlStateNormal];
    }
}

- (void)photoButtonAction {
    UIImagePickerController *imagePicker = [UIImagePickerController new];
    imagePicker.delegate = self;
    [self presentViewController:imagePicker animated:YES completion:nil];
}

- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info {
    self.photo = [info objectForKey:UIImagePickerControllerOriginalImage];
    [self dismissViewControllerAnimated:YES completion:nil];
}

@end
{% endhighlight %}

Imperative programming involves describing _how_ to do something. Reactive programming, on the other hand, involves  describing  _what_ something is, using streams of values. Here's an equivalent implementation, with ReactiveCocoa.

##(1b) Reactive

{% highlight objc %}
//  SignupViewController.m

#import "SignupViewController.h"
#import <ReactiveCocoa/ReactiveCocoa.h>
#import <libextobjc/EXTScope.h>

@interface EditProfileViewController ()

@property (strong, nonatomic) UIImage *photo;

@end

@implementation SignupViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self configureImageView];
    [self configurePhotoButton];
}

- (void)configureImageView {
    RAC(self.imageView, image) = 
    [RACObserve(self, photo) map:^UIImage *(UIImage *image) {
        return image;
    }];
}

- (void)configurePhotoButton {
    @weakify(self);
    [RACObserve(self, photo) subscribeNext:^(UIImage *image) {
        @strongify(self);
        if (image) {
            [self.photoButton setTitle:@"Change photo" 
                              forState:UIControlStateNormal];
            return;
        }
        [self.photoButton setTitle:@"Add photo" 
                          forState:UIControlStateNormal];
    }];
    [[self.photoButton rac_signalForControlEvents:UIControlEventTouchUpInside]
     subscribeNext:^(UIButton *button) {
        @strongify(self);
        UIImagePickerController *imagePicker = [UIImagePickerController new];
        imagePicker.delegate = self;
        [self presentViewController:imagePicker animated:YES completion:nil];
    }];
}

- (void)imagePickerController:(UIImagePickerController *)picker didFinishPickingMediaWithInfo:(NSDictionary *)info {
    self.photo = [info objectForKey:UIImagePickerControllerOriginalImage];
    [self dismissViewControllerAnimated:YES completion:nil];
}

@end
{% endhighlight %}

Our reactive code is more linear than our imperative code, but also arguably more difficult to parse. It's hard to see the value of reactive programming at this level of complexity, but as we add more functionality, we'll begin to see the benefits of choosing ReactiveCocoa.

To finish our form, we'll need a username field, a password field, and a submit button. The submit button should only be enabled when we've added a photo and entered a username and password. For some extra reactivity, let's make each text field's text turn orange while editing.

{% highlight objc %}
//  SignupViewController.h
// ...
@property (weak, nonatomic) IBOutlet UITextField *usernameTextField;
@property (weak, nonatomic) IBOutlet UITextField *passwordTextField;
@property (weak, nonatomic) IBOutlet UIButton *submitButton;
/// ...
{% endhighlight %}

First, let's try updating our imperative implementation from **(1a)**.

##(2a) Imperative

{% highlight objc %}
- (void)viewDidLoad {

    // ...

    [self.usernameTextField addTarget:self 
                               action:@selector(textFieldDidChange:)
                     forControlEvents:UIControlEventEditingDidBegin | UIControlEventEditingChanged | UIControlEventEditingDidEnd];
    [self.passwordTextField addTarget:self 
                               action:@selector(textFieldDidChange:)
                     forControlEvents:UIControlEventEditingDidBegin | UIControlEventEditingChanged | UIControlEventEditingDidEnd];
    [self updateSubmitButton];
}

- (void)setPhoto:(UIImage *)photo {

    // ...

    [self updateSubmitButton];
}

// ...

- (void)textFieldDidChange:(UITextField *)sender {
    [self updateSubmitButton];
    [self updateTextFieldColors];
}

- (void)updateSubmitButton {
    self.submitButton.enabled = self.photo
        && self.usernameTextField.text.length > 0
        && self.passwordTextField.text.length > 0;
}

- (void)updateTextFieldColors {
    self.usernameTextField.textColor = self.usernameTextField.editing ?
        [UIColor orangeColor] : [UIColor blackColor];
    self.passwordTextField.textColor = self.passwordTextField.editing ?
        [UIColor orangeColor] : [UIColor blackColor];
}
{% endhighlight %}

Dealing with state is messy in the imperative world. Our code has become rather difficult to understand — we update the submit button in three different places, `setPhoto:` has three side effects, and in general, things seem to be happening all over the place. Also, notice that modifying our imperative implementation required touching nearly everything we'd written in **(1a)**. Adding functionality usually means adding new events and states. In the imperative world, we have to respond to discrete events and state changes and make sure everything stays up to date, resulting in less linear, more tightly coupled code that's more difficult to understand and update.

Let's try updating our reactive implementation.

##(2b) Reactive

{% highlight objc %}
- (void)viewDidLoad {

    // ...

    [self configureTextFields];
    [self configureSubmitButton];
}

// ...

- (void)configureTextFields {
    RAC(self.usernameTextField, textColor) =
    [RACObserve(self.usernameTextField, editing) map:^UIColor *(NSNumber *editing) {
         return editing ? [UIColor orangeColor] : [UIColor blackColor];
    }];

    RAC(self.passwordTextField, textColor) =
    [RACObserve(self.passwordTextField, editing) map:^UIColor *(NSNumber *editing) {
         return editing ? [UIColor orangeColor] : [UIColor blackColor];
    }];
}

- (void)configureSubmitButton {
    RAC(self.submitButton, enabled) =
    [RACSignal combineLatest:@[RACObserve(self, photo),
                               self.usernameTextField.rac_textSignal,
                               self.passwordTextField.rac_textSignal]
                      reduce:^NSNumber *(UIImage *photo, NSString *username, NSString *password){
        return @(photo && username.length > 0 && password.length > 0);
    }];
}
{% endhighlight %}

Updating our reactive implementation hardly required any changes to what we wrote in **(1b)**, and the resulting code is much more linear. Because state changes propagate automatically, we can define the flow of state rather than responding to discrete events. Even in this simple example, reactive programming has empowered us to write code that's easier to understand and maintain.

We've only scratched the surface of ReactiveCocoa, but hopefully we've given you a more concrete sense of the power of reactive programming. So go forth, compose signals, minimize state, and put away that spaghetti — let's make lasagna ;)

If you're interested, you can check out the complete example project [here](https://github.com/benzguo/ReactiveSignup).
