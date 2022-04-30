---
layout: post
read_time: true
show_date: true
title:  Automate all the things
date:   2022-04-30 10:00:00 +1000
description: Automation tips and tricks on how to get started and when to automate and when not to automate.
img: posts/2022-04-30-automation/hero.jpg
tags: [security,non-technical]
author: Peter Dodemont
---
Learning how to automate tasks has proven time and time again to be one of the best skills I have acquired over the years. The benefits you get when automating are significant. You become more productive as your error rate will almost certainly drop very close to 0; you get to stop doing boring repetitive tasks, instead, you can spend more time on the more exciting and engaging tasks; security is improved as you can limit to access to an account that does the automation. But you might wonder how to get started with it and how to decide what to automate. In this article, I'll explain what my thought process is for deciding how to automate a task, as well as share some tips that were useful for me when learning how to automate.

## When to Automate and When Not to Automate
Deciding when it is worth automating something and when it isn't can be tricky. Especially if you have already automated the most obvious tasks previously. I'll go through the all the steps of my approach first, followed an example of those steps:
The very first step is to determine if the task at hand is recurring (either on a schedule or ad hoc). Next, is getting a full end-to-end overview of what will be required to complete the task I want to automate, not just actions I would do myself, but also actions by other people (I will explain why others are important as well shortly). After I have a list of actions, I will try to work out how long each of these actions takes when done manually. Once I know how long the entire process is, I will determine the frequency of the tasks. You might think it odd that I only determine the actual frequency at this point, but if I were to do it earlier I might decide that a task I do every 6 months is not worth automating because it is so infrequent. That task might take 2 weeks to complete though, so if I know it will take 2 weeks it might still be worth looking into further. Then, I will try and work out what the automated tasks will look like in an ideal world. This then gives me the ability to work out an estimate of how much time would have to spend on the task once automation has been completed. I can then compare that to the total time of the manual task and combine that with the frequency to determine the time saved. There is one final piece of information that I then also include before making a final decision and that is the reduction in errors. Anything done manually is prone to errors, once automated the same actions will be done exactly the same way each time. There will likely still be errors (e.g. errors in the original input, or changes to application behaviors) but the error rate will be significantly reduced.
I don't have a hard and fast rule for when it is appropriate to automate. The way I normally make the decision is that if I believe the time involved to automate can be "recovered" within about 1 year, this includes the time saved by other people on their specific actions.

Now let's get to the example. I'll use a new employee joining the business. Starting once the recruitment process has been completed and finishing as the user logs in for the first time. This is an obvious task to automate, but it is very useful as an example.
First, let's check if this is a recurring task. In every business I ever worked in, there have been people joining that need new accounts, so it's safe to assume that this is a recurring task. It usually is not a set schedule though.
Next are the actions required to complete this task (they are in the general order I have seen them completed in, in the past):
* Filling out the new user form (and in some companies, there might be an approval process here as well)
* Creating the account for the user
* Assigning appropriate licenses to the user
* Provisioning an email address and mailbox for the user
* Assigning the user the appropriate access (usually by adding them to the correct groups)
* Adding the user to any required distribution lists
* Adding the user to any channels/teams in any collaboration apps
* Creating additional accounts for the users in applications that are not using single sign-on with the main account, and setting up the correct access in those apps
* Provisioning a device for the user
* Installing the required apps on the user's device
* Providing the user's credentials to the manager
* Manager provides the user's credentials to the user on their first day
Now that we have the list of actions let's assign the time required for each
* Filling out the new user form and approval if needed - 15 minutes
* Creating the account for the user - 15 minutes
* Assigning appropriate licenses to the user - 5 minutes
* Provisioning an email address and mailbox for the user - 10 minutes
* Assigning the user the to the groups (access, distribution lists, and collaboration) - 10 minutes
* Creating additional accounts for the users in applications - 30 minutes
* Provisioning a device for the user - 10 minutes
* Installing the required apps on the user's device - 30 minutes
* Providing the user's credentials (both the main and for additional apps) to the manager - 5 minutes
* Manager provides the user's credentials (both the main and for additional apps) to the user on their first day - 5 minutes
Then I need to determine the frequency. Since this is not on a schedule, I will need to speak to the service desk to find out how often they need to create new users. For this example let's say there are 10 new employees each month.
The next step is to determine what the automated process looks like and estimate the time effort required for each step:
* New user form is filled out and approved if needed - 15 minutes
* IT receives the form for validation - 5 minutes
* Once validated the filled out details are automatically used to create the user account, assign the appropriate access, distribution lists, and collaboration - 0 minutes
* The user is sent their username and password via separate means (e.g. personal email and SMS) - 0 minutes
* Accounts in additional applications are created and the correct permissions assigned - up to 30 minutes (if they don't integrate usually it will be very hard to automate reliably)
* Credentials for the additional applications are provided to the user (via their corporate email) - 5 minutes
* A device is assigned to the user - 5 minutes
* Device provisioning is initiated and includes installation of the apps required for the user - 5 minutes
The final piece of information I need is a look at the reduction in the error rate. Let's say the error rate is 2 a month in the manual process, and 1 every 2 months for the automated process. For argument's sake, I'll also say all errors need to be manually fixed and it takes 20 minutes to fix each error.
If I combine all the numbers I spent a total of 135 minutes on the manual task 10 times a month, plus 40 minutes fixing errors. this totals 1390 minutes spent each month on this task.
For the automated task, I spent 65 minutes 10 times a month, plus 20 minutes fixing errors. This total is 670 minutes a month.
By subtracting those 2 from each other I save 720 minutes (or 12 hours) a month once the process is automated. Using an 8-hour day, that means that I can spend nearly 4 full working weeks automating and still be ahead within a year. Estimating how long the automation will take has many variables that are highly dependent on the environment, so I won't try and do that here. But I doubt I would need 4 full weeks.
There is also value in the additional security brought by automation that can be taken into consideration, but this is a lot harder to define tangibly unless you have experienced security incidents related to those types of tasks previously. So I will also leave those out of this calculation, but I did want to mention them

## Automation Tips and Tricks
Now that you know the process I follow to make my decision, it's time for some tips on how to get started or implement automation.
* Don't try and automate everything in one go, break it up into smaller pieces that you can test and implement independently.
This way you can start reaping some benefits a lot quicker, and you also won't get overwhelmed with what needs to be done. If we take the same example as before, start by writing a script that creates the user from input you provide to the script (through a file or via the command-line). Once you have that working, move on to adding users to groups to a script. Then, once that works try and combine the two.
* Don't be afraid to make mistakes and own up to them.
I have made many mistakes automating things, they go from small things like accounts not getting created in the correct locations or where details are swapped around to much larger issues where several accounts got deleted including mailboxes or breaking line-of-business applications. In all instances, I put my hand up saying it was probably me because I did this or that. Not once have I gotten into trouble over it, and by being honest the problems could get fixed very quickly and colleagues could assist where needed. I have also learned a lot more from the errors I made than from when everything went right. And I have learned how to test things in such a way that any errors are much more limited in scope now.
* Have a goal in mind.
Personally, I can't motivate myself to learn something if I don't have an end goal in mind. Most of the time the end goal is a real-world problem I am having. I started learning PowerShell because there were tasks I wanted to automate that weren't possible using simple batch scripts. And while, before I had specific tasks I couldn't automate using batch files, I thought many times that I should learn PowerShell, I could never find the will to do it.
* Don't give up.
While some things might seem impossible when you first start, if you stick with it your skill level will increase. Things that once seemed impossible all of a sudden become possible.
* There isn't only 1 right way
There are many ways problems can be solved in scripting and usually, there are several ways to do the same action, each with its own pros and cons. The one to use will largely depend on what you put more emphasis on.
* Review your previous code when you re-use it.
Inevitably you will come to a point where you will re-use entire scripts or sections of scripts. It's always a good idea to review them, not only to make sure they will work, but you might have learned something new or notice a different way of doing what is in the code that you prefer.

This brings me to the end of this article on automation. I hope it will assist you on your journey to automating all the things. As always, if you have any questions or comments please reach out.
