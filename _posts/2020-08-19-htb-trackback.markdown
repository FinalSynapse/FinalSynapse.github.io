---
title:  "HTB Trackback"
date:   2020-08-19 20:00:00 +0930
tags: [HTB]
---

Okay so I did notice this was retired a couple of days ago. I did pop this box about a week ago but I didn't really take notes or screenshots. I didn't realise it was retiring so soon. Having said that this was an awesome beginner box for someone starting out. It kinda teaches you how to do research in real life. The kind of mode of thinking you need to get into. I will try to recall as best I can of how to go about this box. Let's begin.

***

![](https://raw.githubusercontent.com/FinalSynapse/FinalSynapse.github.io/master/images/HTB_Trackback.png)

Ok se we start with a nmap scan as usual. See other write ups and you see what parameters I use stright off the bat. Namely the -sC -sV tags. Immediately we see this box is severing a web page so I check that out. On the site we see a someone talking about leaving a backdoor. Usually this is some kind of reverse shell you will later learn how to essentially inject into machines using various methods. Notice the message is from the person that made the box. At this point you should be doing things like checking the source code of the site as well.

So from the source code we see a comment with that backdoor message. This is where you go start hunting for what that means. So spoilers: This is referencing a tweet the creator of the box made. He has a github so I checked that out and it has a repo of all his tools. At this point it's reasonable to assume that one of these tools is the "backdoor" he is referring to.

Using this list we enumerate the site. I forgot the specific tool I used but this shouldn't be too hard to google. This is something common you should learn. Often you are looking for an admin login page to a site that isn't linked from the main page and you would otherwise need to know the full URL to get there. You should get a hit on one of the tools. Have a look at how that tool works (read the script) and using this tool we get our first bit of access.

So for the next part you need understand how SSH works. I assume you do moving forward. Using the tool one of the things I can access is where fingerprints are stored of users that SSH into the box. Here I created a SSH fingerprint on my kali box and added it to this file. Now we can SSH into the box. At this point you look around see what you can access, I'm not going to go trough everything I tried here, but you need to develop a strategy here for yourself. Using the "sudo -l" command I noticed another tool he left behind. Pretty stright forward to get the user flag from there. :)

So I did mention you should know how SSH works. You should know how computers work. What they are doing inside as they are starting up and as you login. In this case I edited the /etc/update-motd.d/ which dictates what happens as we login, recall that we can SSH into the box at this point. I simply added a line which reads the root.txt file out to me upon login. I close my SSH session and re-login in a new one. As I login just after the welcome message I see the flag displayed. :)

In conclusion this is a pretty easy box. All the tools are already on it. You just need to understand how it all works. And from RL experience that is not uncommon to leverage an admin's clumsiness or 'preference for convenience over security' to pop their box. Security works in layers and often some of those individual layers are not as good as they could be, whether it be because of a time/money issue or the ignorance of the system's caretakers. What the beauty of this box is the real life research we did at the start. This is such an important lesson. At the very least it should make you think about the kind of information you're putting out there, what your publishing on your social media, what information could potentially be used against you. Having said that it's never that obvious or stright forward in real life. The research you do in this lesson based on the hints given might take you an hour or so, in real life you might spend days, weeks or months gathering information and piecing it together. As a security professional you will develop skills close to a detective or investigator, or even that of a spook.

Thank you for you time reading this post. Good luck and have fun!
