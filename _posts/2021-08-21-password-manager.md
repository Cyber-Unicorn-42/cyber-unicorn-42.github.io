---
layout: post
read_time: true
show_date: true
title:  Managing your passwords
date:   2021-08-21 10:42:00 +1000
description: How to manage all your passwords and make them as secure as possible.
img: posts/password_manager.png
tags: [Security,Passwords,Personal]
author: Peter Dodemont
---
Most of us are having to deal with an ever increasing number of passwords in our lives. And while there are lots of smart people out there working on a passwordless future, it seems unlikely that we will ever truly go passwordless. There are multiple reasons for that, but the main reason is that we need some form of authentication that is 100% reliable. And by that I mean that it is not affected by outside factors (see the [biometrics](#biometrics) section for more details).
If I don't believe we will ever go passwordless, how do we manage this ever increasing number of passwords while maintaining the best possible security posture.

## Tools and strategies for managing passwords
There are several ways that we can manage this increasing number of passwords. I will go through a variety of tools and strategies below. I will start with those I think all of us should be using and work my way down to those that should be avoided at all costs.
And finally closing with some comments on items that do not relate directly to password management but are still very important.

### Password manager
This one is the top recommendation I would give to anyone, stop trying to remember all your passwords and use a password manager. With a password manager you can create truly random and unique passwords of any length for each service you use, and you won't even need to remember them. You will just need to remember a single password to get into your password manager. And because you only need to remember a single password you can make this password one that is difficult to guess but easy to remember. See [my guidance below](#passphrases) on how to make one of these passwords.
From my personal experience a good password manager needs a few things.
* It needs to be able to store your passwords in a way that makes it inaccessible to anyone but yourself.
* It also needs to be able to generate truly random passwords.
* It needs to be user friendly and convenient to use. This is not to be underestimated as a difficult to use application will just not get used.
* It needs to be accessible from anywhere. With most of us using our mobile devices for a lot if not most of our online activities it is important you can also easily access the tool from any device.
* Ability to share some password securely between multiple people. This might not be as important when you are single, but once you have a family being able to share accounts, for say online shopping, makes a whole lot of sense.

Over the years I have used and trialed most of the popular password managers both free and paid ([1Password](https://1password.com/), [Bitwarden](https://bitwarden.com/), [Lastpass](https://www.lastpass.com/), [Dashlane](https://dashlane.com), [Keepass](https://keepass.info/) ... ).
All of the password managers I have used served their purpose, these days I use 1Password and it is the one I would recommend. It is very user friendly, works across all devices and includes some great features (like being able to use it for multi factor authentication as well). And while 1Password is a paid solution it is a very worthwhile investment. At the time of this article the pricing was USD $2.99/month for a single user or USD $4.99/month for a "family" of 5. The time and effort that would be required to go through all the online services and reset the password already make it worthwhile. Plus the additional protection of a unique password per service means a compromise on 1 site will not affect any others.
Of that list there is only 1 I really would not recommend you use and that is Lastpass. While it is a capable password manager, [recent research](https://www.theregister.com/2021/02/25/lastpass_android_trackers_found/) has discovered that it contains a large number of trackers. And while there is an opt-out functionality, the additional effort required to opt out as well as the significant limitations they imposed on their free tier users makes this manager a no go.

### Biometrics
Password managers are great for all the online services, but you won't be able to use the password manager until you get into the device it is installed on. And that device should also be as safe and secure as possible. One of the best ways to do this is to use biometrics. When I say biometrics I'm mainly talking about fingerprints and facial recognition (there are also other forms of biometrics like vein pattern or iris recognition but those are not generally available in consumer devices).
Biometrics not only increase the usability, in most cases they are also more secure than a password or passcode as they can't be guessed.
That being said, all devices will still ask you to set up a passcode or password as a backup solution. The reason for this is that biometrics are not 100% reliable. There are numerous factors that can affect your biometrics e.g. if you have been in the water for some time your fingers will prune causing fingerprint sensors to stop working, facial recognition is dependent on the lighting conditions. In those situations, you still need to be able to access your device hence the backup passcode or password. In the next section I will cover what I do when I need to manually create a password that I need to remember.

### Passphrases
While this option is to be avoided as much as possible, sometimes you have no choice but to manually create a password. There are some things you can do to create a password that is easy to remember but hard to guess.
Instead of using a password, I use a passphrase. What I mean by that is that I use a sentence and I write the sentence as one would normally write it along with capitalization and punctuation. The reason for using capitalization and punctuation is a that lot of places still use outdated password policies (see my [previous article](\password-policies.html) for more on that).
The choice of sentence to use is an important one as it needs to be something that is not easily guessed. The strategy I use is to find an uncommon event that happened in my life recently and that is reasonably uncommon e.g. "My son threw a ball that hit me in the face.". Choosing common events that might apply to lots of people would over time make the passphrase less secure as it would become known when there are data breaches in the future that would include that sentence. An example of a bad passphrase would be "It was sunny outside" or in the current pandemic "I had to start homeschooling.".

### Re-use the same password
This should be avoided at all costs. At no point should you have multiple services or devices that use the same password, even if you attach very little value to the service you put those passwords on. If you are using the tools and strategies above you will only have to remember a handful of passwords (1 for your password manager and 1 for each device you use).
Even if the service you have is on a device that doesn't support having a password manager (e.g. gaming consoles) you can open the password manager on your phone and view the password there. While it might be painful most of these devices will have an option to remember the password which I would recommend using instead of choosing a simple password that was re-used. The reason being that most of these services also have portals accessible through web browsers and the devices you are saving the password on are generally not portable and stay inside your house.

### Multi-Factor Authentication
This is something that you should enable on every service it is possible. And while it does not help with the managing of your passwords, it does make it harder for someone to access any of your accounts even if they manage to get a hold of your account.
The preferred method of multi-factor authentication would be app-based. App-based MFA is not as easily intercepted as SMS based MFA.
In general terms to intercept app-based MFA someone would need access to your device or have compromised the device. With SMS based MFA they can have your number ported or redirected by your carrier without needing more than some of your "personal" details.
SMS based MFA is still better than no MFA, but if you have the option of using app-based MFA, you should use that.
Another advantage of app-based MFA is that some password managers can act as the app that provides the MFA. They can scan the QR code shown on your screen and act much the same way as the app would. The added benefit is that you won't need to manually type in or copy in the code, the password manager will fill it for you automatically or with a click.

## Final thoughts
I hope this article has given you some useful information on how you can manage all those passwords and do all you can to keep them as safe and secure as possible. If you think something is missing or have any other comments please reach out to me and let me know.