# 与社会工程师工具包创建者的讨论

> 原文：<https://www.social-engineer.org/social-engineering/a-discussion-with-the-creator-of-the-social-engineers-toolkit/>

![A Discussion with the creator of the Social Engineers Toolkit](img/0443eabe4130bcdea93e19a3cdca180e.png)

因此，昨晚我们与戴夫“Rel1K”肯尼迪和许多有问题的人一起主持了一场精彩的会议。以下是那次谈话的文字记录。

享受并继续关注，因为我们将举办更多这样的会议。

**loganWHD:** so tell us the story of how SET was created
**ReL1K:** Well, long story short when loganWHD came to me and told me about social-engineering.org and what he was going to do, I thought it was a great idea…wanted to contribute where I could..
**ReL1K:** Thought of a way to help augment social-engineering and client-side attacks in a way that 1\. would take down the internet (just kidding), and 2\. be usable by people that really need it..
**ReL1K:** To be perfectly honest when I do social-engineer attacks it takes FOREVER to setup different attack vectors
**ReL1K:** 20+ hours right?
**ReL1K:** anywhere from coding the website
**ReL1K:** to getting the payload delivering properly
**ReL1K:** to setting up email phishing
**ReL1K:** just takes a long time, things don’t work right
**ReL1K:** so wrote SET to automate it, take the crummy parts out of a social-engineering attack and make it more fun
**ReL1K:** plus while drinking it led to a lot of fun times
**Xpl0it:** 🙂
**loganWHD:** love it
**loganWHD:** and
**loganWHD:** it has been one of the heavy hitters of the site
**ReL1K:** so that was it…loganWHD was my inspiration i actually had some ascii pr0n in the image of chris, but that was cut out in the introductions of the SET tool in 0.1alpha…
**loganWHD:** UGH
**loganWHD:** @clean
**SE_Bot:** loganWHD: “clean” is (#1) we try to keep it clean, fun and all about SOCIAL ENGINEERING, or (#2) This is a family-friendly channel and discussions here reflect on our business. Please use appropriate language and discussion topics.
**ReL1K:** I SAID pr0n!!!
**loganWHD:** ugh
**Xpl0it:** geezz…
**Xpl0it:** not a good thought
**ReL1K:** on a serious note, it was a blast writing….
**loganWHD:** so on to better things
**ReL1K:** and was a blast that loganWHD tested it out for me
**loganWHD:** it was awesome
**ReL1K:** without him couldn’t have written it…or had it working
**loganWHD:** still is
**loganWHD:** sometimes i pwn my sons computer just for fun
**loganWHD:** “hey boy open this pdf”
**ReL1K:** hehe
**Xpl0it:** lol
**loganWHD:** shells are fun
**ReL1K:** nothing like getting a meterpreter shell on your sons computer..
**ReL1K:** i tried that but my sons 16 months old
**ReL1K:** doesn’t understand yet..
**davehardy20:** I pwn ed my managers pc with SET
**Xpl0it:** hehehe
**loganWHD:** HAHAH
**loganWHD:** nice
**loganWHD:** so now
**loganWHD:** questions
**ReL1K:** haha
**loganWHD:** there were a few sent in or discussed about SET
**ReL1K:** roger
**ffxp:** I have a quick one after rAWjAW
**davehardy20:** my manager is the IT Manager, he really should know better
**davehardy20:** so I wanna know everything about SET cause its gonna change the way things are done around my office
**ReL1K:** absolutely
**ReL1K:** fire away
*ffxp wondering why I’m trying to be polite*
**ffxp:** 🙂
**ReL1K:** ffxp you can go too 🙂
**davehardy20:** I was wondering if we can make custom websites for the java app attack part of SET?
**ffxp:** for me it’s kind of rare to have the same box initiating the emails or whatever
**rAWjAW:** oh ReL1K… I saw in the newsletter you are talking about the ARP / inject bad things stuff… When you have them go to your site, are you spoofing the DNS resolution, or will it say “172.16.3.99” in the browser… And also, with the wget, aren’t you in a race condition with the computer (even though you are arping it) to get the page and add the payload?
**ffxp:** as the box getting the shells or doing anyalitics
**rAWjAW:** (did that all go through)?
**ffxp:** so I normally have a php box internal or external to handle the metrics / reptoring stuff
**ffxp:** any thoughts about hooking into “collector” scripts or frameworks like beef?
**STS301:** don’t we have meeting now?
**ReL1K:** alright to answer rAwjAW first: right now its only doing it based off of IP, haven’t thought about the DNS route but is absolutely plausible, will add to the TODO list, I like the idea. The second question, we can delay the client as long as we want before receiving connections to the webserver, timeouts are always a worry but shouldn’t be a huge issue, there will be two implementations of…
**ReL1K:** …the wget functionality, one when they are browsing websites, and another to stage ahead of time
**loganWHD:** i love that idea
**loganWHD:** wget
**ReL1K:** to rip a sites front level page takes roughly 4-6 seconds depending on bandwidth, should be easy to rip and pop the site up without a real notice of performance toward the website
**loganWHD:** made me giddy
**ReL1K:** and based on python, the basehttpserver is highly optimized and multi-threaded, shouldn’t notice to much lag rendering
**loganWHD:** pop a box
**rAWjAW:** sounds good 🙂
**ReL1K:** to answer ffxps: totally understand, do you want me to add a field to just create a listener automatically for you on a seperate machine? like menu 1-4 = X menu 5 = setup a metasploit listener?
**ReL1K:** because currently you can setup a remote host automatically through the payloads
**ffxp:** not sure what the best way is, just wanted to get an idea where your head was at regarding this
**ReL1K:** davehard20: the new functionality with the wget should absolve that, i can add an option to point to a folder that can be imported to a custom website, thats super easy
**ReL1K:** **ffxp:** typically if your using two separate machines, one for emails the other for a listener, you can create the payload, shoot out the emails, but making sure the remote host is set for your other server, SET gives you the option to not create a listener
**davehardy20:** that sounds like great solution add a pointer
**ffxp:** ok
**ReL1K:** **ffxp:** can easily add in a method to just create a listener
**ReL1K:** give me 3 minutes
**ffxp:** its mainly that I get analytics rather than pop shells
**ffxp:** 🙂
**ffxp:** or rather 🙁
**Xpl0it:** lol
**Xpl0it:** get your smiley faces straight
**Xpl0it:** 😉
**ffxp:** maybe I can add in some php/py/asp code to listen for some standard analytic payloads
**ffxp:** that would go along with the kit
**ffxp:** brb…got to put kids to bed
**ffxp:** errr…pwn them via SET
**ReL1K:** there we go
**ReL1K:** just added a number 5 on SET for create a payload and listener
**ReL1K:** you can use that to create the payload you want, and just setup the listener on the other machine
**ReL1K:** just comitted the changes
**ReL1K:** should make it a little bit easier at least
**ReL1K:** ffxp, i can add a simple addition
*loganWHD updates*
**loganWHD:** SWEEET
**ReL1K:** use the windows/upexec/reverse_tcp
**ReL1K:** so you can specify your own executable
**ReL1K:** could be something trivial
**ReL1K:** for statistics
**ReL1K:** instead of a shell
**ffxp:** mmmm
**ReL1K:** what do you use it for?
**ffxp:** to show how easy users can be tricked …then report on the depth of their stupidity
**ffxp:** thats the analytics part
**ffxp:** and sometimes report on how ineffective end point controls can be
**Xpl0it:** nice
**ReL1K:** are you looking for more of like a report of how many people connected?
**ReL1K:** or something thats not harmful
**ReL1K:** at all
**ffxp:** exactly the kind of stuff I’m doing now with php
**ffxp:** right
**ffxp:** I’m trying to find the code now
**ffxp:** stupid simple
**ReL1K:** how about
**ffxp:** just writes to a file
**ReL1K:** this is for the email attack only right?
**ffxp:** yes
**ReL1K:** you don’t want it to connect back to you?
**ffxp:** correct
**ReL1K:** just create a file?
**ReL1K:** oh
**ReL1K:** SUPER easy
**davehardy20:** I’d be interested in a report on how many people connected
**ffxp:** gather info
**ReL1K:** that would be windows/exec
**davehardy20:** this would prove how thick some users are
**ReL1K:** execute ipconfig :: “moo.txt & net view :: moo.txt & gpresult :: moo.txt & net user :: moo.txt”
**rAWjAW:** Another question… Right now it is just adobe pdf files, are there plans for other as well? (or a way to select one)
**ffxp:** cool
**ReL1K:** that work?
**ReL1K:** **rAWjAW:** plans for adding word and excel right now, what other formats were you interested in?
**ffxp:** well..I’ just use javascript/java to fingerprint the browser
**rAWjAW:** word and excel were the ones i was thinking of
**ffxp:** then track if they click links
**ffxp:** then see if they download a non-weaponized exe/pdf/doc
**Xpl0it:** **ReL1K:** what about adding PowerPoint?
**ffxp:** thats where the windows/exec thing would be cool
**ReL1K:** **Xpl0it:** shouldn’t be to much harder..
**Xpl0it:** k
**ReL1K:** **ffxp:** i gotchya
**ReL1K:** i haven’t looked at ppt yet
**Xpl0it:** k
**Xpl0it:** just thinking about what stuff people usually e-mail
**ffxp:** thx man
**ReL1K:** no problemo
**ReL1K:** will add
**ReL1K:** anything else guys?
**ReL1K:** betting ill have the wget portions done in 2 weeks
**loganWHD:** ReL1K, thank you
**ReL1K:** dns spoofing is easy
**ReL1K:** probably tomorrow
**loganWHD:** thank you making the tool
**loganWHD:** and thank you for your time tonight
**ReL1K:** hehe tis fun
**Xpl0it:** yeah great GREAT stuff
**ReL1K:** love this stuff 😉
**_Elwood_:** looking forward to that wget stuff. Great stuff!
**rAWjAW:** yes yes, thank you very much for SET ReL1K
**rAWjAW:** oh
**ReL1K:** _Elwood_ i love you buddy!
**ReL1K:** hehe thanks rAWjAW 🙂
**loganWHD:** and from the downloads on the site… i know alot fo ppl thank you
**loganWHD:** haha
**Xpl0it:** lol
**rAWjAW:** I know these aren’t typically e-mailed but what about an exe embedded in a .chm file
**rAWjAW:** I have a writeup somewhere on how to do it
**ReL1K:** send it away, can always add the option
**rAWjAW:** Sure thing
**ReL1K:** alright guys much appreciated
**loganWHD:** l8r bro
**ReL1K:** “<3” loganwhd
*Image*
*[https://unsplash.com/@carlevarino](https://unsplash.com/@carlevarino)*