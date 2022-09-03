---
layout: post
read_time: true
show_date: true
title:  Design a network your future self will not curse you for
date:   2022-09-02 10:00:00 +1000
description: How to design a network simply, effectively and ready for what the future might bring
img: posts/2022-09-02-designing-a-network/hero.jpg
tags: [security,networking]
author: Peter Dodemont
---
Over the years I have come across many different network layouts. There is one thing most had in common, they were all overly complicated. Most of them were also not very future-proof, as expansions or unexpected additions were not built into the design. Which caused me to redesign the network during my time at that particular company. In this article, I will cover what I have learned over the years, and the principles I follow when designing the network these days.

## The Foundation

![A poured cement foundation being leveled, in close up](/assets/img/posts/2022-09-02-designing-a-network/foundation.jpg "Foundation")  
Just like any good design, having a solid foundation is extremely important. The foundation needs to be able to support everything that is built on top of that and keep it safe and secure.  
In network design, security is achieved through segmentation. With segmentation, you divide the network into different segments and control what traffic can flow between each segment. Segments themselves can then also be divided into more segments, sub-segments if you will. And those can be divided again, and so on and so forth until you have the number of segments you need. This means that your foundation needs to be able to accommodate many segments.  
Beyond segmentation, we also want to have a network design that is easy to remember and use. If you constantly have to search the documentation to find details of the devices you are trying to connect to, you will be losing a lot of time when troubleshooting issues. Normally, anything I design is self-documenting, but in the case of networks, this isn't really possible. Instead, I aim to design the network so that you can fit the documentation on a single page.  
With these things in mind, let's get started with building the foundation, by choosing which private network to use. I always use the 10.0.0.0/8 private network address range, using this range gives me immense flexibility. I have seen other people use the other private ranges (172.16.0.0/12 and/or 192.168.0.0/16) but they always end up being hard to remember, unable to expand upon when something unexpected comes up, or become complicated when needing to expand.

## First Building Blocks

![small cuboid blocks in different colors, stacked like a pyramid, in close up](/assets/img/posts/2022-09-02-designing-a-network/buildingblocks.jpg "First Building Blocks")  
Now that I have the network range I am going to use, I need to decide on how to divide it up. For this, I list the different locations that require network access within the business. This includes all the offices, but also any data centers or other locations where there is equipment that requires access to any resource on the network.  
I use the 2nd octet of the address range for this part of the segmentation. I normally use a /16 subnet for each location. This works for up to 256 sites if there are more than that, smaller subnets will need to be used (this hasn't happened to me yet). /17 will give you 512 subnets to work with, /18 will give you 1024, and so on. But be careful not to make the subnets too small, otherwise you will not be able to implement some or all of the items I will talk about later.

## Main Construction

![blue steel frame of a house, small child sitting in front of the frame ina yellow t-shirt holding a small stuffed green dinosaur](/assets/img/posts/2022-09-02-designing-a-network/mainconstruction.jpg "Main Construction") 

Next, I need to make decisions on how to further segment the network at each different location. Part of my philosophy was simplicity and fitting all documentation onto a single page. To achieve that, I design a template that can be applied to each site.  
Since I create a single template for each site, I need to collect some information about each site, namely the different device types. I don't just need the types that exist at all sites, I also need the ones that might only exist at some or a single site (e.g. servers).  
Once I have all the device types that exist across the business, I need to find out what the highest number of devices of a particular type is in any single site. I can then use that number to decide on the subnet size for each of the different segments. I will go through an example below that will provide more details on how I do the sizing of the segments (since it is easier to understand with an example).  
In addition to the types of devices, I also need to know if there are special use cases for some devices (e.g. broader access or remote access).  
I use the 3rd octet of the ip address to create these segments.

## Finishing

![white kitchen bench with blue stool chairs in front of it, white overhead cupboards, marble splashback](/assets/img/posts/2022-09-02-designing-a-network/finishing.jpg "Finishing") 

The final part of the design is assigning a standardized 4th octet, there are 2 parts to this. First, there are devices that exist in fixed numbers at all or most sites, these will have a fixed 4th octet. Then, there are also ranges that can be assigned to devices, if they vary in number at the sites.  
Having a fixed 4th octet makes it easy to construct the IP addresses for the standard devices at each site, even when I only have limited information.  
I usually re-use the list of devices from the 3rd octet, plus some additional information on how some of the network devices might be configured (e.g. some devices might be in a cluster for high availability).


## An Example and Creating My Documentation

To build the documentation, let's look at an example that ties this all together. Here are the basic details I will use as a starting point.  

- 5 offices
- 2 datacenters
- Some servers are in the cloud
- the largest office has 500 people

Going back to the foundation, I always use the 10.0.0.0/8 network. No real need to document that as it generally won't be used anywhere.  
Next are the different locations, in this example, there are 10. The 5 offices, the 2 data centers, and the cloud are obvious. There is also going to be a VPN location for devices connecting in remotely, and I usually also like to have a single subnet that stretches between the data centers (this makes moving servers, as part of failover or otherwise, a lot easier). 
As per my design, I use the 2nd octet to divide the private network into /16 subnets, 1 for each location.  

![A table with names in one column and ip addresses in the next column, each octet of the ip address has a different color, and the 2nd octet changes for each line](/assets/img/posts/2022-09-02-designing-a-network/2ndoctet.png "2nd Octet")  
As you can see from the screenshot above, I simply list the sites and then increment the 2nd octet by 1 for the next site. I then leave a large gap and assign the VPN and data center network subnets near the end of the available range, finally, I add the cloud even closer to the end of the range, again leaving a gap between the end of the data center ranges and the cloud.  
I don't create a particular order for the sites, I have tried to use many different schemes to order them in the past, but it inevitably stops following that scheme as the business opens or closes offices. So instead I just pick an order at random. The gaps I leave between site ranges, VPN range, data center ranges, and cloud ranges are there in case of future expansion. The business is more likely to open more new office locations than they are new data centers. And there are likely to be even fewer cloud providers than there are data centers. Hence the large gap after offices and the smaller gap after datacenters and an even smaller gap after cloud providers.  

Now I need to work out what all the different device types and what is the highest number at any single site.

- Corporate user devices - Max 500
- IT user devices - Max 5
- Printers - Max 10
- Routers - Max 2
- Switches - Max 25
- Wifi access points - Max 10
- Firewalls - Max 2
- IP phones - Max 500
- Meeting room PC's - Max 5
- Meeting room TVs - Max 5
- VM's - 200
- Physical Servers - 10
- UPS's - Max 1

As you can see there are a lot of different types of devices used across the business. Let's use that to create the template for each site.  

![A table with names in one column and ip addresses in the next column, each octet of the ip address has a different color, and the 3rd octet changes for each line](/assets/img/posts/2022-09-02-designing-a-network/3rdoctet.png "3rd Octet")  
You'll notice that there are fewer subnets than there are device types. The reason for that is that I group devices together into different categories, depending on the network access required.  
- Corporate is for PCs and mobile devices issued by the business, and also meeting room PCs (all these devices require access to corporate resources).
- Voice is for phones
- Guest is for devices that do not need corporate access (e.g. TV's so that they can perform firmware updates or visitors)
- Printers is for any network printers
- Management is for management interfaces on networking equipment (i.e. routers, switches, wifi access points, firewalls, UPS's and physical servers)
- IT is a special network for IT workstations as they will generally require different levels of access than other devices (e.g. ability to connect to other sites)
- VPN is for any user devices connecting in from outside the internal network
- Servers is for servers in the corporate network
- DMZ is for any servers exposed to the internet

I choose my subnets and their size in the following way. I will start by ordering them from the largest category of devices within a site first, and work my way down to the smaller ones. If certain subnets only exist at one or a few sites, these go at the end of the list (the VPN, servers, and DMZ in my example here).  
Then I need to decide on the size, this is where the maximum number of devices comes in, I add all the numbers up of devices that sit within a category. I then pick a subnet that will be large enough to accommodate that plus at least 50% more to account for any future growth. I then leave a gap big enough to move to a larger subnet before repeating the process for the next subnet.  
In my example, there are 505 corporate devices (the user devices plus the meeting room PCs), but there will also be some addresses for firewalls and maybe routers (depending on how it has been set up), so that totals about 510 devices. Using a /23 could work (I know that a /23 will accommodate 510 devices, but there are many ip subnet calculators online you can use if you don't know this off the top of your head) but, that leaves no room for growth. So instead I chose to use a /22. This can have up to 1022 IPs which leaves us with plenty of room. I then leave more room to grow the subnet to a /21 just in case something unexpected comes up. This might not be that I need to grow the subnet to a /21, but it could be that PCs and mobile devices need to be separated out or even something else I haven't thought of. Leaving that room gives me flexibility.  
I repeat this process for each subnet until I reach the /24 subnets, for these ones I don't leave any gaps as that doesn't really serve any purpose since that is the smallest size subnet one can have using the 3rd octet.  
Once I have done all the subnets that exist at most sites I start with the subnets that only exist at one or a few sites. For these, I just pick a subnet further along and assign that, no real method other than something that is easy to remember and has room to expand.  
I also usually assign IT the 42 subnet, since that is the answer to life, the universe, and everything.
{% include youtubeplayer.html id="tK0urw144cU" %}

The last part is assigning the 4th octet. As mentioned I will just re-use the list of devices from the 3rd octet and include some additional information from the configuration of some of the devices.  

![A table with names in one column and ip addresses in the next column, each octet of the ip address has a different color, the 4th octet changes for each line, and the last 8 lines include ip ranges](/assets/img/posts/2022-09-02-designing-a-network/4thoctet.png "4th Octet")  
The choice of what values to assign to the 4th octet is entirely up to you. Personally, I like using high numbers for the standard devices (like the routers and firewalls in my example), other people prefer to use low numbers, it really doesn't matter.  
You can see that I have again left some gaps, which you will know by now is just in case something comes up. I then also kept the addresses for the routers and firewalls following the same pattern to make it easy to remember.  
For subnets with a varying number of devices, I prefer to use DHCP scopes as that means the devices can then just be added by anyone without needing to be configured in any special way (most devices come with DHCP set as the default method for getting an IP address). If these devices require static IPs (like printers or management interfaces) you can always assign a DHCP reservation. This also helps with replacement devices as you can just update the reservation without having to manually configure something on the device.  

When all put together the documentation looks like this  

![a spreadsheet combining all 3 previous tables next to each other from left to right](/assets/img/posts/2022-09-02-designing-a-network/full.png "Full Spreadsheet")  
If I ever need to refer to the documentation, I can just pick the item from each section as I go. The color coding also makes it easy for any other staff that might need to use the documentation, even if they are not familiar with the terminology.  

That covers the process I use when having the design or redesign a network. I will leave out how to implement this or what to watch out for when implementing a new or redesigned network for a future article. As always, if you have any questions or comments please reach out.