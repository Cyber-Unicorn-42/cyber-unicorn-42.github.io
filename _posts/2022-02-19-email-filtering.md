---
layout: post
read_time: true
show_date: true
title:  Email Filtering Do's and Don'ts 
date:   2022-02-19 10:00:00 +1000
description: My tips on what to do and what not to do when you are looking to do email filtering.
img: posts/2022-02-19-email-filtering/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
Let's cover off some of the common configuration mistakes (in my opinion at least) people make when they setup email filtering, and why I believe they are mistakes. I will also provide the way I believe email filtering should be setup. Some of the points will go against "common" knowledge, but I hope that from the explanation I provide you will be able to understand why I say this.

## Allow and Block Lists
This is probably the number one "mistake" I see people making. Just blindly adding any email address to a block list when spam is received and conversely blindly adding email addresses to allow lists when requested by people high enough up in the business. Both of these are terrible practices in my book, and I would not employ either a block or an allow list as they both have their own flaws.

Let's start with block lists. These might have been useful in the past when malicious content was sent from randomly generated email addresses. These days a lot of the malicious emails come from real email addresses that have been compromised (either directly or because the device is compromised). Don't get me wrong, there is still malicious email from fake email addresses out there, but these can be caught out in different ways. Because a lot of the email addresses used you will inevitably end up with a situation where a legitimate email is blocked because the email is on the block list (this has happened at nearly every business I worked that used block lists). Which usually causes significant frustrations as getting and NDR becomes complicated by having to use external email addresses (e.g. from Google) or having to dig through logs.

On the other end you have the allow lists. I dislike and discourage the use of these as they would allow a bypass of significant parts of your email protections. And as I mentioned previously, a lot of malicious email comes from compromised email addresses, there is no way for you to know if that email address will not be compromised in the future. Even if it is safe now, there is no guarantees it will tomorrow.

The only exception I make to these rules is if there is an automated system talking to another automated system via email (e.g. alerts being sent from a system to a ticketing system to log a ticket). As there is no end users involved the risks are lower and the sporadic nature of large burst tend to trigger SPAM filtering policies.

While I don't have a direct replacement solution for block and allow lists, they should not be needed as the focus should be on detecting malicious and legitimate email through all the other options at your disposal.

## Prevent Spoofing of Your Own Domains
Spoofing is when a sender tries to send an email from a domain they do not control or are not allowed to send from. So, to prevent spoofing SPF, DKIM and DMARC records can be used to identify systems that you have authorized to send email from your domains. If you don't know what SPF, DKIM or DMARC are, I have an article [here](\SPF-DKIM-DMARC.html) that explains all of them in a way that doesn't require any technical knowledge.

Preventing spoofing doesn't just protect your internal employees by ensuring only authorized system can email from your domains, it also provides this ability to anyone else that might receive an email from your domain.

If you wouldn't prevent spoofing, a malicious actor could pretend to be the CFO or send an email with a malicious password reset link, and there would be no way for the person receiving the email to determine of that email really came from the CEO or the identity management system.

## Impersonation Protection
You might think that impersonation and spoofing are the same thing, but they aren't. While not preventing spoofing would allow a malicious actor to impersonate someone, it's not required. With spoofing any email address existing or not could be used. Impersonation protection also goes beyond just looking at the email address.

With impersonation protection you can prevent a malicious actor from sending emails pretending to be someone within the company. Usually this will be a c-suite or manager, but it can just as well be someone from the service desk or anyone else for that matter.

Some of the things impersonation protection can look at (depending on your email filtering solution are):

* Similar internal domain
* Similar monitored external domain
* Newly registered domain
* Reply-to and from addresses mismatch
* Display name matches internal user
* Email address similar to internal user

The similar internal and external domain items will usually not only match with domains that are slight variations of your own domain (e.g. security-pete.com instead of securitypete.com) but also where characters might have been replaced with similar looking characters (e.g. replacing the letter o with the number 0 or using a Unicode character).

Newly registered domain speaks for itself, these are domains that were only just registered and are already sending you email a few days later. This is highly unusual as most business will take time to reach a point of wanting to use their own domain and getting a new domain setup for use with email is usually a process that takes more than a few days in an established business.

Reply-to and from addresses mismatch is just what it says. It compares the reply-to address with from address from the email and compares them. It should do this across the from address in the header as well as the from address in the envelope. If you are wondering what all these addresses do, here is a short explanation. The header from email address is the email address that sent the email to the server, the envelope from is the address that is displayed in the email client, this does not have to be the same as the header from. A good example of this is marketing newsletters, these are usually sent from mailing services (e.g. Mailchimp, SendGrid, AWS SES ...) so the messages are sent from an email from those services and that is in the header from. The from address displayed in the email client might same marketing@securitypete.com so this is different to the actual email used to send from. The reply-to address is what is used when you hit the reply button, in most cases this matches either the header from or the envelope from.

Display name and email address matches for internal users are also pretty self-explanatory. Traditionally these were only configured for very specific people in a business, but I like to enable this for everyone.

If you were to go and trigger the impersonation event as soon as any of these were detected, you would get a lot of false positives and a lot of frustrated users. So instead, you need to set it to trigger when multiple items were triggered for a given email. Finding the right balance of what items to trigger on and how many is going to be a process. I will usually start with enabling all items and triggering when 2 items are detected. I will then monitor what happens and if there are too many false positives, I will look at what item triggered them, if a very large percentage was triggered because of a particular item I will disable it, if there is no clear item causing it I will up the count to 3. I haven't had to disable more than 1 check or up the count beyond 3 to get to a point where there are virtually no false positives anymore.

False negatives can be equally problematic, but these are harder to have a general solution for. In most cases, it will be targeting a stricter policy to the specific user that was successfully impersonated. If you are lucky and they have a fairly unique name, you can set a policy to trigger on their name alone. Otherwise, you will have to investigate why the mail got through and act accordingly.

## URL Protection
We all know that malicious actors love to send through emails with URLS to phishing pages or other websites that host malicious content. URL protection is what we can prevent these links to cause any issues.

URL protection usually has 2 main stages to it. First, the domain will be checked to see if it a domain that has already been reported or marked as malicious. If the first check passes than an automated system will "click" the link and analyze whatever is displayed or downloaded. If no malicious content is found the email will be let through.

When first introduced links were only ever checked upon first receiving the email. Malicious actors quickly found a way around these systems by not having any content present on the site for the first little while after the email was sent, and only load the malicious content once they believed most of the emails had passed through the email filtering and into the users' inboxes.

Email filtering solution caught onto to this and nowadays URL protection doesn't just check the links when first received, but also checks them when the user actually clicks the link. They do this by replacing the URL in the email with one that point to their systems that do the checking. This provides much better protection as now even if the content is loaded later, it will still get checked before the user will be able to access it.

This more advanced protection does pose a problem for the traditional advice that users should hover over any link to check the URL. Because the URL is now replaced it is not as easy for the user to check. Personally, I don't believe this is a great loss as reading URL's is difficult even for tech-savvy people, so checking the URL was never really a great solution, even if users were doing it.

If you don't believe me have a watch of the below video from Emily Schechter on this topic.
{% include youtubeplayer.html id="UD-ukjVoeLc" %}

## Attachments
Similar to URLs, attachments are also commonly used by malicious actors to compromise users or devices. There are many different types of files that will be sent through, most dangerous file types (like executables) will be blocked by default, but there are some to pay special consideration to. Encrypted and unreadable archives ("zip" files) should also be blocked. The reason being that there is no way for you to know what the content of this archive might be as it cannot be scanned. I know more and more businesses are using encrypted archives, because they require a password to open. From a sender's perspective that makes them more secure, but on the receiving end they entail a lot more risk. My recommendation here is to share the files from within the archive via a file sharing service that allows you to set a password on the link. This still gives the security of a password to access, but will allow scanning of the files for malicious content by security solutions from the file sharing provider.

Encrypted documents can also be used instead of encrypted archives, these usually don't rely on a password, but instead rely on being able to prove you are a valid recipient of that file. These are lower risk for now, as malicious actors don't have infrastructure in place to mimic this behavior, but in time I'm sure that will change.

Beyond protections from the email side, there are also things you need to do within applications to protect your users. It's impossible to block Microsoft Word and Excel files as both are used extensively. Both of these can be very dangerous as well because they support macros. Macros can be made to do just about anything the user can. So preventing macros from being enabled or requiring them to be signed digitally is an example of how to increase the protection for files that can't be blocked.

## SPF, DKIM and DMARC
As mentioned in a previous section DNS, DKIM and DMARC are used to prevent spoofing of domain names. This means you should also be checking these for any incoming emails. This way you can be sure that the email has come from a legitimate service. These checks are universally enabled by default. No further configuration should be needed, but it won't hurt to double check them.

## Greylisting
The last of the items I will mention is greylisting. Greylisting is a technique used to try an identify emails from malicious infrastructure. Most (if not all) legitimate email services will retry to send any messages that failed sending because the receiving end reported a transient error, while malicious infrastructure usually doesn't bother with this. Greylisting does exactly this, it will send a transient error message with a request to retry sending the message a few minutes later. This happens twice with the message finally being accepted on the 3rd retry. In general, this is only ever done, whenever a new system is detected as sending email for a particular domain. The first or first few messages from that domain will be greylisted and asked to retry. Once the system has passed the greylisting test, any future messages will just pass through without needing to pass this check again.\

## Dealing With Requests to Relax the Rules
I quite regularly will get requests to relax rule x or y. My answer to these requests is always starts with no. It usually stays no as well, there are rare occasions where it will become a yes, but I have to be given some very good reasons. It is my job to provide the best possible protection I can, while not hampering the normal business operations. Now you might wonder how I can get away with saying no to request that come from all levels of the business, and the answer is that I explain to people why these rules exist in terms they can understand. This works for the large majority of cases, because once users understand the reasons, they usually understand the risks a lot better.

For those cases where this does not work, it always helps to bring in someone who has more authority (says a c-suit exec) who is on your side. Even if the user still disagrees with you, they are less likely to push back if they see you have the backing of an executive.

There is a lot more I could talk about on email filtering, but I think these are the most important topics to cover. You might also have noticed that I haven't explicitly said what actions to take in what situation. This is because this highly depends on what the business wants, they could be ok with blocking everything, or maybe they will want an administrator to review some or all of the alerts. So, this is something you will need to discuss internally. I mostly worked in medium size businesses, with lean teams, and almost everything will either be blocked or allowed. Nothing is manually reviewed unless requested by the end user.

As always if you have any questions or comments, please reach out.