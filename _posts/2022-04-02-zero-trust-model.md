---
layout: post
read_time: true
show_date: true
title:  The "Zero Trust" Model
date:   2022-04-02 10:00:00 +1000
description: What do I think the zero trust model is and how to get started.
img: posts/2022-04-02-zero-trust-model/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
Zero Trust is one of the most used buzzwords in the information security space at the moment. So, you have probably heard it thrown around by quite a bit. Unfortunately, there is set definition for zero trust, everyone that uses it gives it their own spin and so it can be very confusing to actually work out how you can get started with zero trust. I want to bring you some information on what I understand under zero trust and how I think you can get started with it.

## What is Zero Trust
As I mentioned zero trust means different things depending on who is saying it to you. What I understand under zero trust is, that every single time someone needs to use an application or service their identity gets verified. Let's break that sentence down into the separate components and explain each one.
### every single time someone needs to use an application or service
There are many services people use for example logging in to a device, opening the mail client, logging in to a cloud portal, opening a line of business (LOB) app. And each of these should be considered in isolation from the others.  
The reason for requiring each to be isolated is that there is no way of knowing what has happened between the last event and this one. For example, someone could have taken over the device remotely, the user might have left their device unattended and a stranger is using it.
### their identity gets verified
An identity is not just a username and password, an identity is the combination of everything the system knows about this person trying to access the service. This means an identity would include the OS or browser that is usually used, the time of day, the username and password ... .  
Because the identity includes a lot more than just a username and password, verifying the identity doesn't equate to asking them to enter their username and password.

## Getting Started with Zero Trust
Now that you know what I understand under zero trust, how can one get started with implementing zero trust.

### Identity Provider
When choosing your identity provider (IDP), you will need to evaluate which services they are able to integrate with and what data can they collect. You will want it to integrate with as many services/applications as possible. In general, you'll also want it to collect as many pieces of distinct information as possible. The more pieces, the better the identity will be (within reason off course, gathering someone's height or weight isn't going to be useful for the purpose of verification an identity when they log in to a cloud service). Some of the pieces of information that I believe should be collected are the compliance status of the device (e.g. is it encrypted, is it a corporate device, does it have up to date anti-virus software ...), normal working hours, application/service usage patterns (e.g. the payroll application only gets used every 2nd week on Thursday), location and time of each verification ... . This is by no means an exhaustive list as it varies between IDP’s, but it gives you a good idea of what can be used.  
I also recommend sticking with a single IDP, as management will be a lot easier, and the profile they will be able to build will be much more comprehensive.

Your IDP will collect quite large amounts of data on your users and their behaviors, this makes choosing one a very important decision as you must trust they will not misuse that data. 2 of the largest IDP's out there are Microsoft Azure AD and Okta, and no while no company is immune to being breached (both Microsoft and Okta were breached recently) they are both well respected. There are plenty of other well respected IDP's out there as well. I just mentioned those as they are the most well-known. And for a lot of people Microsoft Azure AD is something they can leverage at a pretty low cost as they already have other services with them.

An additional bonus you will get from moving to a single IDP is single sign on (SSO). Meaning users will only need to remember a single set of credentials across all services. And depending on the policies configured might not even need to provide credentials at all.

As a side note, not all IDP's are focused on the corporate environment. Facebook or Google are identity providers that consumers can use (you have no doubt seen the sign in Facebook or Google buttons on websites before). I won't cover the consumer IDP's in this article as they generally don't incorporate with a lot of corporate focused services.

### Risk Profiles
Another key items to consider when going down the zero trust route is understanding the how many different risk profiles you have and what they are.
I find that the easiest way to determine risk profiles is to look at what would happen if a malicious actor were to get access to a system. Let me give you a few examples.
* Payroll System: this is generally high risk as it gives the ability to add new "employees" on the list that would get "paid" at every pay run. Thus, causing immediate financial loss, but could also lead to money laundering or other charges.
* Mailing list Service: this could be medium to high as sending out a newsletter with malicious content would cause reputational damage.
* Internal training tool: this would low risk, as it might only give access to take courses or training.
Going through and thinking about the risk profile for each service will be very helpful in determining how to handle identity verification for those services.

When evaluating the risk profiles, I recommend you start with a number-based system rather than words as it will give you more options to create nuances between each application. Also, when evaluating your first application or service don't give it the highest or lowest rating. Since this is the first one you can't know if there will be any that or worse of not as bad down the track. If you have a very large number of applications or services, focus on the ones that are being used most frequently.  
After you have created the list, get it validated by someone else and explain to them why you gave the rating you did when asked.

### Policies
With the IDP and risk profiles you will be able to start crafting all the different policies. The policies would not just be based on the risk profiles though, there could be other considerations that need to be taking into account.

One very important aspect I take into consideration, but that is usually ignored is the user experience. The user experience needs to be the best it can be. A lot of the time when the user experience is poor, the user will look for a workaround, not because they have malicious intent, but because they want to get on with being productive. Once a workaround is in place most of time, you not only lose the additional security from implementing this less user-friendly implementation over the more user friendly one, you also lose any additional security from a more user friendly implementation. The reason for this is that the workaround will bypass the entire process.  
We can take MFA as an example. If we force the user to use a phone call to their desk phone as an MFA solution, this is not the best user experience we can provide. It will take some time for the call to come through, the code might be misheard, need to be at the desk ... If instead we allow the use of SMS notification to their mobiles, these usually can't be misread, can be received anywhere ... There is generally less ways this can go wrong for the user providing a better experience, but they are easier to intercept and abuse. But because they are so easy users are less like to try and work around them. An even better option in push notification or one-time passcodes as these are even more convenient and they are more secure as well.

Other than user experience, frequency the application gets used might also be something you look at for example. If we take the MFA as an example again, you can force MFA on every sign in for the user, but the user will get tired of these extremely quickly and find workaround. They could leave their session logged in indefinitely, or just start accepting MFA prompts regardless because it has become a habit. A better user experience is to limit the MFA prompts to specific scenarios and use all the other information in the identity as a form of additional factors.

Again, this is not an exhaustive list, as the factors you look at are something you will need to determine for yourself. And they could change over time as well.

The policies themselves can also contain a variety of options. Most of the time they will come down to the same basics though.  
First, validating what application is being accessed and thus what risk profile is applying. Once you know the risk profile you can evaluate any other consideration that apply. This is then finally followed by details on how the identity verification will take place.  
The validating what applications are being access is usually performed based on the URL used to access the service, or some identifier being passed in the authentication request. The other factors will vary depending on what you are evaluating. You might even do any actual checks, but rather group your applications together if they have the same additional factors and thus can rely on the first step by itself. The identity verification can be whether the user will be prompted for MFA or use other factors to provide that MFA or require the use of a compliance device.
All these options will vary depending on the IDP you are using.

As mentioned, it doesn't have to be based on a risk profile at all. If for example you only have staff in a single country, you can create a policy that forces MFA if there are any attempts from outside that country, or even flat out block them. You are only limited by the option that you IDP provides to you.

### Deployment
Once you have your policies set up, it will be time to deploy this to the end users. There are a myriad of ways to tackle this. And choosing which one to use is something you will need to do yourself. I will list the 2 methods I generally use.

The first method is, starting with a handful of test users. Usually these are users within the IT team, as they are more open and there are easier lines of communication. I will leave the policy on for these users for a week or two so they can report back any issues they experience, and adjustments can be made.  
After this we move on to a representative group of users across the business (usually between 20 to 50). Most of time I don't communicate to these users that they are part of this test, as I want to get honest feedback. Once you tell people they are in a test group, you will usually get them blaming everything that goes wrong to the test, or they over report things they would usually not be bothered with. When to communicate and when not to communicate is something I do "by feel" and comes after many years of trial and error. The service desk would usually be informed though so they can escalate any issues through.  
The final stage is companywide implementation. By this stage most issues will have come to light already, which means there will be very little/if any in the way of adjusting the policy.  
This process provides not only a smooth process for implementation, it also keeps the workload on the service desk team and myself manageable.

The second method is, turning the policy on in audit mode. This should give you the expected outcome for each time the policy has applied so that you can tweak it if needed. I use this for policies that have a very limited number of users that will have the policy apply to them where it is not possible to use the first method. Audit mode can generate a lot of information you will need to go through, this is very time consuming and error prone. But when only a handful of users are involved the impact of not being able to log in is usually significant, hence taking the additional precautions. And it also provides a better user experience.

Choosing which services to start with can be tricky. I will usually start with a service only used by the IT team. This is to get the IT team familiar with the process. This way when we get to rolling it out across the business, they know what it looks like in a day-to-day usage scenario.  
After I have done one or more IT exclusive services, I go for companywide services that have a low impact (e.g., a training application). This provides the users with an example of what this process will look like in their day-to-day activities.  
After one or two of those, I will move onto the services that are in use on a daily or near daily basis. These services will get provide the greatest benefit from a security perspective.  
Finally, I move into the limited use application, starting with the ones that have a high-risk profile. By this stage all the users will be across what it looks like and what to expect so the process becomes very smooth.

## It's a Journey
I want to add a final note here to make it clear that implementing what I understand as zero trust is a journey that will take a very long time. There are lots of services that integrate with IDP's but there are just as many if not more that don't yet. But even if you can't implement it across all the services, being able to deploy it across some is a good start. And as you progress you can start adding integration with the IDP as a requirement for any new services you sign up to or getting the internal development team to add this into your application (if you have custom in house applications). Benefits like SSO don't only improve security but also the user experience and thus other areas of the business will want to take advantage of those benefits as well.  
All that to say, don't be discouraged if you won't be able to implement it across the board. Do it where you can and take it one step at a time.

As always, if you have questions or comments please reach out.
