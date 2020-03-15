---
draft: True
title: "16 year old app developer | Plagiarist or Prodigy"
description : "Harshita Arora 16 year old App Developer Plagiarism Crypto Price Tracker"
date: 2018-02-05 11:50:27 +0530
tags: [Uncategorized]
---

**Update : Sun Feb 11 09:10:07 IST 2018**

* The post was temporarily taken down to protect the interests of people involved. A lot of them received threatening messages , and so did their employers

* Names of people who contributed to this post have been withheld to protect their anonymity.

* After the Daily Beast article, saying that we took down the post because the information was false, we decided to put it back up, because the information is not false, and we would like to stand our ground.

* Some people can't let things die FFS!

---

**Final update : Tue Feb  6 17:19:56 IST 2018**

So I think I have said and done all that I can in this matter. I've locked comments (you guys can't keep conversations civil and I had to keep moderating too often, thanks for the awesome discussion people!). I'll leave this post up as I stand by all that has transpired.

This remains an exemplary example of how sad human behavior is, how easily people deceive you, how frauds get away with anything and everything, and how ; when push comes to shove ; people do not mind throwing you under the bus.

Lots of lessons learnt, lots of amazing view points encountered.

PS - If anyone wants to discuss further on this matter, please head to Hacker News and Reddit, this won't be available as ground of discussion anymore.

---

**A sad update : Tue Feb  6 12:14:30 IST 2018**

* I was woken up by a frantic call from my friend at 7am, inquiring about an email he saw on a [reddit-comment](https://www.reddit.com/r/programming/comments/7vjibv/email_from_harshita_16_yo_dev_threatening_to_sue/) ; in case the image is unavailable, you can find it here : [image](https://i.imgur.com/PrDjiyo.png) . Last night she had tweeted to the same person's employer : [tweet](https://twitter.com/aroraharshita33/status/960605288557559808)

* The said person allegedly issued an apology on his and my behalf because he was really, really scared. About this, I got to know from this [comment](http://disq.us/p/1pxju7u) ; image just in case : [image](https://www.dropbox.com/s/amcxkjhpukouceh/Screenshot%202018-02-06%2012.19.01.png?dl=0)

* The said person issued an apology under duress, in which he shifted the blame for his own misdeeds on people, painting a very bad picture for everyone.

It is very , very sad that the internet, which used to be a safe space to voice our opinion, can be weaponised against you by making such bizzare allegations. It has been tiring two days with constant moderation to keep the discussion civil, to catch up with people who have been affected by allegations and threats per se. I cannot oversee all the comments people make on reddit and hackernews, can only account for my blog. But it pains me to see brilliant people from industry being threatened and their employers being harassed just because they voiced their opinions freely.

---

So I don't usually do this, but given recent turn of events wrt the Crypto-Price-Tracker app which made headlines all across the internet, I am inclined to write this post.

A few days ago a [crypto price tracker app](https://www.reddit.com/r/Bitcoin/comments/7tl7gi/im_16_and_i_made_an_app_that_will_help_you_trade/) was launched on the App Store and got massively popular owing to the fact that a 16 year old girl claimed to be the "maker". Remarkable work for a 16 year old, I agree. She gained a lot of appreciation from CEO of Product Hunt, and from esteemed developers across the globe. Here’s the announcement if you missed : [Press release](https://medium.freecodecamp.org/today-i-launched-my-first-mobile-app-heres-what-i-learned-6fc25c14eee6)

But after reading her blogpost, her stackoverflow/github history in an attempt to understand her struggles as a self-taught programmer; I figured that there was no struggle at all! Self taught programmers are full of numerous questions, but she didn’t seem to have any! Neither is there any progression of all the tasks she did.

As a full time dev myself (over 4 years of work experience), I know the amount of effort that goes into getting a prod ready app going. Claiming to do that in 2months with camera-ready pitch is suspicious to say the least. Something was amiss, so I decided to look further. I used a spare iPhone to get the app, and then used https://github.com/BishopFox/bfdecrypt and https://github.com/BishopFox/bfinject to decrypt the app .

Here are a few things I found :

* No, she did not code the app as she claims. In her defense, she has **help** from a few people, but I would call that bluff, given that help construed of over ~~50%~~ 90% of git commits.

* ~~There is just ONE storyboard in the whole app. ONE!!!~~ There is no storyboard, lol. For any iOS dev who is just starting out, making an application without storyboards is just unbelievable.

* After converting the .nib files to readable format, I found the name of primary developer who probably got no credit for building a marvelous app, whereas she is being lauded as a the next global prodigy.

* Now, a common way to cover up plagiarism in apps is to replace bundle ID with your own. But, traces are sometimes left in code. So we did a grep for `com` on strings in the decrypted app, you will notice that on line 929 there is a mention of library which does not exist, and so I’m inclined to believe, this is who probably the author of original code is.

* Upon reaching out to the the dev in question, he went to the extent of saying that he "allowed her" to call the app hers, given certain "circumstances". Fair enough, we don't see him bagging MIT scholarships, or all the accolades the girl is receiving for her app. Let's pass on some appreciation to the so-called-mentor?

We tried raising our voice on reddit, twitter, facebook but the were quickly hushed. Our posts were deleted, accounts flagged and blocked, all because we called out a person on her dishonesty and work ethics. So here is another, albiet last attempt at making a difference. If after this, my github/blog/reddit/twitter - anything gets blocked, you know who the real bully is.

* Here is the facebook post : [Facebook](https://imgur.com/a/jFbQR)
* Reddit post that got removed : [Reddit](https://webcache.googleusercontent.com/search?q=cache:zla9eLCXxQcJ:https://www.reddit.com/r/Bitcoin/comments/7v7l3k/all_about_that_cryptopricetracker_prodigy_or/+&cd=1&hl=en&ct=clnk&gl=in)
* Her confession on chat with a colleague : [Messenger](https://imgur.com/a/Dj5yT)
* The git contribution graph : [Git](https://i.imgur.com/s4HojFd.png)
* For strings, refer here : [Strings](https://pastebin.com/XkqnvFpK)
* For the ipa and decrypted payload : [IPA](https://www.dropbox.com/sh/wmstu6jv3wsn5j3/AADuTWCzQCnkHSStcXCDr8P4a?dl=0)
* The public defense of ghost developer : [Facebook](https://i.imgur.com/kISYRpp.png) , [Archive](http://archive.is/sTMy1)
* The message he sent when we made the reddit post, calling them out : [iMessage](https://i.imgur.com/IuhtQv0.png)
* More threats : [1](https://i.imgur.com/ij840Ll.png) , [2](https://i.imgur.com/0ihzIzy.jpg) , [3](https://i.imgur.com/TQzc6TW.png)

And no, before someone gets on the bandwagon and calls me a bully, or says I'm harassing her, I am not. This is me simply stating the facts. She had become the torch bearer of women-in-code. I don't agree with that. Such kind of people give **Women in Code** a really, really bad name, because ~~sweetie~~, it takes tonnes of hardwork, cutting through competition, persistent dealing with sexism and lot of patience to make place for yourself in this male-dominant coveted industry. Being a poster girl for women in software, without knowing how to do a decent job at same coupled with bad work ethics is not right! Period.

As much as I have no issues with someone being an excellent entrepreneur, it isn't fair to pass off work done by a hired contractor as your own and claim fame for it. I personally have no problems with her outsourcing the app development either, but I do however have a problem with her **not** owning up to it, **not** crediting the app as `team work` and claiming that she coded the entire application.

Even if she was just an architect of the application, I believe she should claim herself to be just that, and not the primary app developer. A lot of companies/entrepreneurs hire freelance devs to build initial prototypes and there is nothing wrong with that. But claiming fame as a [16 year old developer who ideated, designed, coded and marketed the app under 3 months](https://imgur.com/a/f2aGw) is a little far fetched and really discouraging to those that attempt to teach themselves programming. Software dev is plain hard work. Prodigious powers not required, IMO.

Developing computer software is complicated business, and yet, a huge number of youngsters are staking their claim writing apps that take the world by storm. Has app development really become accessible to all? Are we living in an age of prodigies?

I wish her good luck for the future, and hope she owns up to the fact that she isn't the only one deserving of all the glory she has lately been basking in.

