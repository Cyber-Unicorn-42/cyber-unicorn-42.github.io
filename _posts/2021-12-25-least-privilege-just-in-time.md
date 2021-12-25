---
layout: post
read_time: true
show_date: true
title:  Least Privilege and Just-In-Time.
date:   2021-12-25 10:00:00 +1000
description: What are least privilege and Just-In-Time and some tips on getting started with them.
img: posts/2021-12-25-least-privilege-just-in-time/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
While both Least Privilege and Just-In-Time (JIT) are relatively simple concepts to explain, implementing them successfully is not quite as easy. The road to fully implementing both across an entire business is generally quite long. Luckily you don't need to implement everything in one go, you can start small and build upon it over time.

## Least Privilege Model
The thought behind the least privilege model is pretty simple:
##### Only provide just enough access for a user to do their job
And while the concept is simple, implementing this definitely isn't. It usually starts with figuring out what permissions are needed for every role in the business and ensuring that they don't provide more access the needed. This process can be pretty difficult and long, and it is also not a once and done process. It will need to be continuously monitored to make sure there is no new permissions for a role, permissions can be removed from a role, or even if there is a role that needs to be added or removed.
Then you also need to have regular reviews of each user, to account for permissions that were added in an ad-hoc fashion.

## Just-In-Time Access
JIT is an equally straightforward concept:
##### Only provide access for the duration that access is required to perform the given task
This might be best explained with an example, let's say a user needs to modify a page on the intranet, they should only be given access to modify that page just before they make the change, and it should be taken away again as soon as they are finished.
The way the process usually works is the user requests the access needed and specifies a time period they need it for (think hours, not days). Then the permission would be approved and if granted it would be applied and then removed again after the requested time period has expired. While it would be possible to do the who process manually, automation is going to be key for this. Going into detail about automation us beyond the scope of this article, but I will be writing an article about automation in the future.

## Implementing Least Privilege and Just-In-Time
Now that you have an understanding of what both these concepts are, we can start taking about implementing these. And while both can be implemented independently, I think it makes a lot more sense to implement them together. Combining least privilege and JIT makes it so that if an account was to get compromised the damage that could be done would be limited in scope, but the user is still able to do the full range of tasks they need to do.
It is impossible for me to provide you with a detailed roadmap of how to implement these in your organization, but what I can do is give you some information on how I approach the implementation and some examples from my past.
I usually start by asking a number of questions around the particular permission I am reviewing or adding. These questions will help determine my recommendation for JIT as well as applying the least privilege concept.
I'll start by listing the questions and then going into more detail on each question.

* Does the user need this permission for their day-to-day activities?
* Can this permission cause an outage?
* Can this permission lead to compromised users or devices?
* Can this permission cause reputational damage?
* How many people could be impacted if this permission was abused?

##### Does the user need this permission for their day-to-day activities?
This is usually not a question you can answer yourself, so you will need to ask the user. If you ask them if they need it straight up, they will just answer with yes though. My approach is to ask them how often they perform the task that requires this access. I can get lots of answers to this question, so here are some general insights on the possible answers. If the answer points to a high frequency (think every day, multiple times a week), then I will need to rely heavily on the answers to the other questions. If the frequency is low (think once a month or "just in case"), then almost certainly JIT is going to be the recommendation. Permissions for low frequency tasks are a great example of combining least privilege and JIT as you never know what exploits might be discovered in the future and limiting permissions for every low frequency action protects you from these potentially being exploited.
##### Can this permission cause an outage?
##### Can this permission lead to compromised users or devices?
##### Can this permission cause reputational damage?
##### Can this permission cause data loss?
I'll group these 3 questions together as they can generally all be answered by gathering the same information, namely the exact details of what actions will be possible if this permission is given. This can be quite clear from the permission name or description, or completely opaque if you have no knowledge of the application in question. The key task here is to gather as much detail about the permission as possible, and then evaluate the information against each question. Some examples of permissions that would cause you to answer yes to these questions:
Granting a user the ability to update an internal application - This could cause an outage on that application if there is a bug or an incorrect procedure is followed.
Granting a user the ability to bypass content filters - This could cause user compromise by them visiting a phishing website.
Granting the ability to sent bulk email to external people - This could cause reputational damage as once an email has been sent externally it cannot be recalled.
Granting the ability to upload documents - This could cause someone to upload valuable IP to an external service so they can retrieve it from a personal device.
If the answer to any of these questions is yes, then JIT should almost always be applied, even if this is an action the user does frequently. Since the potential impact is significant, we should limit any possible abuse if the account were compromised in the future.
##### How many people could be impacted if this permission was abused?
With this question I try to determine what the impact would be if anyone were to abuse the permission. The answer to this question should flow naturally from the information gathered to answer previous questions. This can be from a single person not being able to access a file or folder, to the entire company not being able to log in because the password of every AD account has been reset.
I don't have a fixed number I use to determine when to use JIT. It will largely be based on the actual impact to the business as a whole rather than just a raw number. For example, causing an outage on the training platform isn't going to have the same impact as an email outage. Both outages affect the entire business, but the business can function perfectly fine for a few days without the training platform, but not without emails.

Let's look at a few examples that have come up in my past.
##### Permission to access all customer contact information
This permission came about in a newly developed internal app, it combined all contact details into a single pane. The app also gave the ability to retrieve details of multiple customers at once. While the users didn't contact every customer every day, they did contact customers every day.
There was no risk of outages or compromise, but reputational damage was a possibility. But since the users already had access to this information, albeit in a less convenient way, the risk of reputational damage wasn't any larger than before. Data loss was a concern as it allowed for easy collation of all the customers in a branch. So, if a user were to resign to start their own competing business, they now had easy access to a list of potential clients. Just like with reputational damage this wasn't previously inaccessible information, but it did provide for much easier access to it.
Finally, the number of people that would be impacted was every external customer.
Ultimately the app was redesigned to limit access on a per branch basis. There were a few users who still requested access to all branches which I recommended against because they only wanted it "just in case". The business deemed the time it would take to provide them access in those situations would be too long and opted to provide these people permanent access to all information.
##### Office 365 global admin access
This permission exists in every single business that uses Office 365. The answer to if this permission is needed on a day-to-day basis is almost certainly no in every single business. On the other hand, in a lot of small and medium sized businesses the answer to is this being used daily is probably yes. This is because it is easier to grant service desk staff global admin right then to actually work out what access they need in Office 365 and only provide that.
The answer to the questions if this can cause outages, compromise, reputational damage or data loss is yes. Because there is no higher level of access in Office 365 it can easily cause all of those and it can cause it for every account in the business.
So, for global admin access JIT is a no-brainer. But even more important than that is the fact that only the most senior people need this level of access. Everyone else should only be granted the permissions required to do their roles (and those should all also use JIT). This is a perfect example of somewhere to apply least privilege and JIT.
##### Provide local admin rights on user's own device
While it is common knowledge these days that it's not best practice, I almost always see that the IT team has this access on their own device. And while it is inevitable for some of the IT team to have this access, that access should be on a separate "admin" account and not their main account.
They almost certainly don't need to install applications very frequently, and if we take out any of the company provided applications (e.g., through Intune/MEM or SCCM), that frequency drops even lower.
Could admin rights on a device for an IT team member lead to an outage, compromise, reputational damage or data loss. Most definitely. They could install an application that floods the network with traffic, install ransomware and are provide higher risk access should the device be compromised.
As for the potential impact of abuse, that could be as severe as a complete business outage.
The answer to each of these separately would be enough to recommend JIT. In this case I would go even further and recommend to not provide this access at all to any user account. Only a select few accounts would be allowed to install applications and those account should not be what the IT team use to log in to their devices. And I would still recommend the use of JIT for these accounts as well.

Now that we have explored the reasoning I use to determine if JIT is appropriate and how I can apply the least privilege model, I want to explore some considerations for the implementation of a solution.
##### Approval and Justification
Every just in time access request should include an approval and justification. Now in some cases this approval can be automatic, but it should still exist. You might wonder why you would even put in an approval if it is automatic. It adds an additional barrier for any malicious actor to bypass. In most cases, they will not know if the approval is automatic or not, so they have to weigh up the risk of requesting approval and possibly being discovered. Approvals can also be used in SEAM solutions to detect anomalous behavior e.g., an approval in the middle of the night and appropriate action can be triggered.
The people responsible for the approval will also vary and it could be a manager or a peer. E.g., on the service desk I would generally give the rest of the team the ability to approve certain JIT requests if they form part of the service desks day to day activities.
##### Time Periods
When users request JIT access, they usually specify a time period they need the access for. Limits needs to be put in place to prevent this access becoming de-facto permanent. These limits should be determined by how long it would typically take to perform the task and/or how often that task needs to be done in a specific period. E.g., someone on the service desk would not be required to ask for permission every time they need to reset a password, they would ask for that permission once and it be granted for a set amount of hours even though the task only takes a couple of minutes. But because they have to this repeatedly throughout the day, having to request it every time would cause a lot of lost productivity. On the other hand, a payroll officer processing everyone's pay once a month would need to request this access each time they performs the task.
##### Automation
As mentioned previously automation is going to form a key part of implementing this successfully as no business has the resources to implement this effectively if everything needs to be done manually. Once the approval has been received, applying and removing the permission should not require any further user interaction. A notification needs to be sent to the user as well to inform them it has been applied.
##### User Access Reviews
This can be painful to run regularly, but when you do the first one, spend time automating the process so future ones become a lot easier. Also try to put yourself in the shoes of a regular business user who is now asked to complete this and make it as simple and clear as possible. I plan to write a separate article on doing user access reviews in the future.
##### Start Small
Pick one role to apply this to at a time. I usually start with the IT team as they are more open to change and some of the highest risk users.
##### Lead by Example
I always start by implementing anything I want onto myself first. The nature of my job has made me very open to change and I am also very security conscious, so if I can't "live" with something end users definitely can't.

I hope this gave you some valuable insights into how you can go about implementing least privilege and just-in-time access.
And as always, if you have any questions or comments, please reach out.