---
layout: post
read_time: true
show_date: true
title:  Password Policies For The Current Era
date:   2021-08-07 11:09:20 +1000
description: What should password policies look like in this day and age.
img: posts/2021-08-07-password-policies/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
We all know what a safe and secure password is right? It should have at least 8 characters, contain at least 1 uppercase, 1 lowercase, 1 number and 1 symbol, it should change regularly and you can reuse an arbitrary number of your previous passwords. But does this actually make passwords more secure?

## Why the current password policies don't work
Let's break down each of the rules that are commonly in place.

### Passwords should be at least 8 characters
This one is actually not bad. 8 characters gives quite a wide variety of possible combinations. Longer is better but 8 is generally enough.

### Passwords should be complex i.e. 1 uppercase, 1 lowercase, 1 number and 1 symbol
This rule has been proven over and over again to not work. The reason for this is fairly simple, we all substitute the same characters. And why do we all replace the same characters because they are visually very close. We also tend to fall back on things we already know, so most will put a capital at the start and a punctuation at the end.
Let's take the [classic](https://xkcd.com/936/) "correct horse battery staple". If you were to ask random people to make this "secure" you will find  that they will do one or more of the following:
* Capital the C of correct
* Capital the first letter of each word
* Add a exclamation mark or dot at the end
* Replace the o's with 0's
* Replace the s's with 5's
* Replace the e's with 3's
* Replace the a's with 4's

(My guess is you probably thought about those exact rules.)
As you can see, we are not as clever as we think we are. We all have the same ideas. Malicious actors are well aware of these substitutions and as such will also do the same substitutions when attempting to guess passwords.

### Passwords should be changed regularly
Just as with the complexity rule, password expiry has been proven over and over to not work. Most people will simply add a number to the end or beginning of their password and then just adjust the number when it comes time to change your password. And again there are a few very common strategies people use:
* Increment the number by 1
* Use the number of the month
* Use the number of the month plus the year (either 2 or 4 digits)

Again these strategies are very trivial for a malicious actor to account for. If they manage to get a hold of one of your passwords (e.g. through a data breach) they will notice the number and will be able to work out the pattern pretty easily.

### Passwords should be different from previous passwords
As I described in the previous point the most common strategy is to adjust a number at the end or beginning to change your password. This doesn't really constitute changing the password as a malicious actor can easily see the pattern and now only needs to guess 1  character (or at maximum a few characters) of the password. This might give someone a false sense of security that their password is safe when in reality it is likely less secure than if it was never changed.

## What should I do instead?

Fortunately there has been lots of research in this area. And my recommendations below are based on what all that research says.

### Passwords should be at least 8 characters
As noted before this is a pretty good rule. Research done by [Microsoft](https://docs.microsoft.com/en-us/microsoft-365/admin/misc/password-policy-recommendations?view=o365-worldwide#requiring-long-passwords) has shown that when passwords get too long users will start using repeating patterns. These long passwords would be easier to guess than shorter passwords that do not repeat.

### Passwords don't need any complexity
Since most of us employ the same substitution techniques, complexity serves no real purpose. It doesn't make the passwords any harder to guess, but it does make them harder to remember. This is cited in virtually all current password guidance documents from entities that have done password research. NIST notes the below in their [guidance](https://pages.nist.gov/800-63-3/sp800-63b.html#appA):
>composition rules are commonly used in an attempt to increase the difficulty of guessing user-chosen passwords. Research has shown, however, that users respond in very predictable ways to the requirements imposed by composition rules. For example, a user that might have chosen “password” as their password would be relatively likely to choose “Password1” if required to include an uppercase letter and a number, or “Password1!” if a symbol is also required.

Microsoft and the NCSC also make similar remarks in their guidance which you can find [here](https://docs.microsoft.com/en-us/microsoft-365/admin/misc/password-policy-recommendations?view=o365-worldwide#requiring-the-use-of-multiple-character-sets) and [here](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach#tip5-password-collection)

### Passwords do not need to expire regularly
As we saw previously users do not change their whole password when they are forced to change it. This can give a false sense of security that the password is safe even if there has been a data breach because it was changed recently. The reality is that the new password is less secure as it can easily be guessed. The password [guidance from Microsoft](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach#tip5-password-collection) says the following:
>Password expiration requirements do more harm than good, because these requirements make users select predictable passwords, composed of sequential words and numbers which are closely related to each other. In these cases, the next password can be predicted based on the previous password. Password expiration requirements offer no containment benefits because cyber criminals almost always use credentials as soon as they compromise them.

The NCSC provides very similar [guidance](https://www.ncsc.gov.uk/collection/passwords/updating-your-approach#tip4-password-collection) as well.

### Use banned password lists
Rather than enforcing arbitrary rules on password you should be using a list of passwords users cannot use. This list of banned passwords should include a corpus of previous breached passwords as will as any words that refer back to the business or website that users are setting their password for. For example if your company name was Security Pete and you were based out of Sydney Australia, you would also ban the use of the words "security", "pete", "sydney" and "australia". These words would be easily guessable if they were used in a password. Microsoft even considers this to be [the most important](https://docs.microsoft.com/en-us/microsoft-365/admin/misc/password-policy-recommendations?view=o365-worldwide#ban-common-passwords) requirement for passwords:
>The most important password requirement you should put on your users when creating passwords is to ban the use of common passwords to reduce your organization's susceptibility to brute force password attacks. Common user passwords include: abcdefg, password, monkey.

NIST offers a similar [recommendation](https://pages.nist.gov/800-63-3/sp800-63b.html#appA).

### Only change Passwords when they are suspected to have been compromised
While there is no direct mention of password history in any of the guidance. Common sense dictates that if your password never expires you will not have to force the password to be different to previous passwords. What you should instead be doing is monitoring new data breaches and comparing the passwords in those data breaches to the passwords of your users. If a user has been found to have a password that is the same as in the data breach you should force a password change onto the user. [Have I been pwned](https://haveibeenpwned.com/) is an excellent free service you can use for checking if passwords or users were in a data breach.

### Use Multi-factor Authentication
While this is not directly a password policy, it is an integral part in the password lifecycle. Using multi-factor authentication gives you additional security even when the user's password has been compromised. And gives you the ability to verify a users identity when asking the users to perform a password reset.
From personal experience having to provide multi-factor authentication every single time you authenticate becomes extremely time-consuming especially if it happens multiple times in a day. As such I recommend implementing risk based multi-factor authentication. Risk based multi-factor authentication uses a number of factors from the user's session as the additional factors rather than prompting the user for the additional factor. It only prompts the user for the additional factor if the session factors indicate the user or the session is high risk. These session factors generally include things like time of the day, current location, is it physically possible for the user to have travelled between their previous location and their current location, are they anonymized (e.g. Tor browser, VPN) ...
While risk based authentication does not provide the same level of security as prompting the user for a factor each time, it improves security significantly while also reducing user friction significantly.

## Final thoughts

The recommendations I put forward here should be implemented together. If you chop and change between them you run the risk of actually reducing the security of the passwords instead of improving them.
I have referenced Microsoft, NIST and NCSC a few times through this post and I will post the links to their respective guidance pages below once again. I do suggest you read through them all as they contain a lot more valuable information than what I have included in the post.

[Microsoft's Password Policy Recommendations](https://docs.microsoft.com/en-us/microsoft-365/admin/misc/password-policy-recommendations?view=o365-worldwide)  
[NIST's Digital Identity Guidelines](https://pages.nist.gov/800-63-3/sp800-63b.html)  
[NCSC's Password Administration For System Owners](https://www.ncsc.gov.uk/collection/passwords)  