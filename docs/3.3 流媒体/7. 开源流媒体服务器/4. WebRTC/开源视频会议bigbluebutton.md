- [开源视频会议bigbluebutton](https://www.cnblogs.com/ustcyc/p/3571407.html)
- [远程教育平台 BigBlueButton ](https://www.oschina.net/p/bigbluebutton)
- [自建在线会议系统 ---- BigblueButton 详细安装步骤](https://blog.51cto.com/hsbxxl/2478440)



BigBlueButton 是一个使用 ActionScript 开发的在线视频会议系统或者是远程教育系统，主要功能包括在线PPT演示、视频交流和语音交流，还可以进行文字交流、举手发言等功能，特别适合用在网上教学，支持中文等多种语音。

界面非常漂亮：

![img](http://www.oschina.net/uploads/img/201003/04141143_BliP.png)

这是另一个开源视频会议项目，简称bbb

官方网站：http://bigbluebutton.org/

代码地址：https://code.google.com/p/bigbluebutton/

demo：http://demo.bigbluebutton.org/

收集到的中文资料：http://www.iteye.com/blogs/subjects/yangactive

支持安卓：http://bigbluebutton.org/2011/02/09/bigbluebutton-on-android-phone/

IPad：http://bigbluebutton.org/2013/01/11/html5-client-ipad/

根据bbb进行二次开发的项目：https://github.com/fuji246/django-bigbluebutton，国人对项目的贡献：http://www.bbbforum.com/forum/topic/61/

关于性能（bigbluebutton的CEO针对一个说bbb性能不好的人的回击）：https://help.instructure.com/entries/22478390-What-are-technological-limits-to-BigBlueButton-

I can give you another update on BigBlueButton's capabilities available to Instructure customers.  Again, **I'm speaking as CEO of Blindside Networks**, the company that started the BigBlueButton project. As of last week,  Blindside Networks now provides a free tier of BigBlueButton to all  Instructure customers.  This free tier provides

- - recording of your sessions (recordings automatically delete after 14 days)
  - latest version of BigBlueButton (improved usability)
  - higher capacity hosting (unlimited sessions)

There is nothing you need to do.  Instructure has upgraded all Canvas accounts to this newer version of BigBlueButton.  For details on the  usability improvements compared with the previous version, see the  following two videos:

  Moderator/Presenter Tutorial (9m):

   http://www.youtube.com/watch?v=PHTZvbL1NT4 

  Viewer (Student) Tutorial (4m):

   http://www.youtube.com/watch?v=LS2lttmPi6A

 

\> I'm curious to learn more about BigBlueButton and any possible limitations it has as well.

**建议一个会议室只有50人，我们没有设计为一个会议室100或者150人以上。**

We recommend using BigBlueButton for sessions of 50 users or less.  This is not a hard-coded number.  You could have 51, 52, etc. in a  session and it will still work fine, but it's our way of saying we've  not designed BigBlueButton for a webinar-type collaboration in which  100, 150, or more.

**bbb是可以多对多视频通话。**

Unlike a webinar where only one person can talk, with BigBlueButton  you can have 20 people all talking, all sharing the webcam, all  collaborating at once.

**多个在线会议室可以同时进行，没有任何限制。**

There is no limit on the number of simultaneous BigBlueButton  sessions that are active.  (Go ahead and try!)  For the upgrade, we've  invested a significant amount into dedicated resources to support the  use of BigBlueButton by the Instructure community, and, we can add more  resources as the community's use grows. 

\> We already know that it doesn't do recordings

BigBlueButton now archives your session for later playback.  Here is an example recording (best viewed by FireFox): http://goo.gl/A0IOXr 

\> I'm especially curious because I was told by Instructure tech  support a while back (late 2012/early 2013) that BBB can only support  200 simultaneous users or less across your entire Canvas LMS

**现在，bbb支持单房间50人在线，房间数目不限制。**

That was then, this is now: we can support up to 50 users in a single session.  There is no limit on the number of simultaneous sessions.  For example, you can have six sessions of fifty users for a total of 6 * 50 = 300 users on-line.  

\> And if more than 25 users are simultaneously accessing BBB is that the audio will be awful! 

Our capacity for hosting BigBlueButton sessions is *far* ahead of  what it was in late 2012/early 2013.  I think the ARM was trying to be  conservative, which.  

**视频不清晰的缘故是宽带太低，而不是bbb的问题。每一个用户需要最少1M的下行宽带和0.5M的上行宽带。**

The quality of audio depends almost entirely on each user's  bandwidth: if the bandwidth is too low, the audio will suffer.  We  recommend that users have , at a minimum, 1 Mbits/sec download and 0.5  Mbits/sec upload bandwidth.  These numbers are guidance as well (they  could have 0.49 Mbits/sec and it should work well); however, the lower  the bandwidth, the more likely the user will report poor audio.

On the administration side, with any deployment of any on-line  classrooms, we would recommend you holding a test session with students  ahead of the real class.  Students (and sometimes professors) will need  help becoming familiar with the setup of their microphone.  The above  videos will guide them through adjusting the audio settings. 

\> "*As a rule of the thumb, without knowing anything about your server, we recommend you use BigBlueButton with on-line classes  totaling twenty-five (25) users or less. This may be one class of 25  users, 2 x 12, 5 x 5, etc.BigBlueButton was never intended to scale to  200-300 users, but rather to help on a classroom level."*

Now you can have up to 50 users in any one session, and you can host  as many sessions as you want.  For greater clarity, we do not recommend  going above fifty users in any single session, so 200-300 users in a  single session is out, but six different sessions of 50 users each is  fine.

Why?  Blindside Networks has a load-balanced architecture that  supports a pool of BigBlueButton servers.  Our hosting environment  starts each session on a different server (usually the least leaded) and spreads out the load across the poll.  As the demand increases, we add  more BigBlueButton servers to the pool. 

\> "Currently no message will be displayed. Product is looking at  what we should do as an error message. Now they will probably get a  message saying they are connecting to the server and it will either die  or say connecting forever. Know that it is on our road map."

There is currently no hard-coded limit on the number of users.  We  didn't want the 51, or 52nd users to not get in.  If you wanted a  hard-coded limit, we could certainly look at implementing it. 

\> Needless to say, I was stunned! We have 25,000 students and  thousands of courses each term, so clearly we could easily exceed that  limitation any given day & we have been steering people away from  BBB ever since. 

Gabrielle, I hope the above gives you a better comfort on the effort  we've put into providing you a solid, usable, real-time collaborative  environment for Colorado Mountain College.  We've been working almost  fourteen months on this new version of BigBlueButton, and we focused on  the stability, usability, and features that our customers have  requested.

**我希望你也能理解，我们不可能满足每一个人的需求。我们建议的三种使用场景：一对一的课程、人数少的多对多在线交流，一对多的直播(50人以下)**

I also hope you can see that we're specifically not trying to make  BigBlueButton do everything for everyone.  The three use-cases we focus  on are one-to-one, small group collaboration, and one-to-many (under 50  users).  

**bbb已经在社区推广一年多了，已经为五万多个课程提供了技术服务。据我们的调查，这些课程中，平均都是11个人，每个课程不超过77分钟（一对多，直播模式）；平均5个人，50分钟左右的多对多视频。**

Since BigBlueButton was introduced to the Instructure community over a year ago, there have been over thirty thousand sessions.  When we  looked data for sessions over 5 users, the average meeting size was  about 11 users and the average length was 77 minutes (one-to-many), and  for sessions containing 2,3,4, or 5 users, the average size was 3 users  and the length was fifty minutes (small group collaboration).  

 Warm regards,... Fred Dixon

And, we strongly recommend that anyone who will be broadcasting audio us a headset.

.  This is integrated with Canvas Conferences. 

September 29, 2013 16:31

 

视频服务器：基于red5。

client：前端可以用flex。

特性： 是一个使用 ActionScript  开发的远程教育平台，主要功能包括语音，视频讲课，桌面共享，在线文档的展示，如ppt,word,pdf等等，voip，还支持多国语言，文字交流，非常合适网上教学。服务器端用到的项目包括有 ActiveMQ,Asterisk,Nginx,Tomcat等！ 

目前版本是8.0，服务器端运行在Ubuntu 10.04 32-bit 或者 64-bit. 部署bbb服务器端有两种方式，一种是：从安装包安装，一种是安装bbb虚拟机！

 

![img](https://images0.cnblogs.com/blog/96437/201402/271405149236523.jpg)

 

**与openmeeting的区别，**

　　国人的一片文章

http://blog.sina.com.cn/s/blog_51396f890102es5b.html，

国外的两篇：

https://github.com/mainehackerclub/open_vehicle_tracker/issues/8

BBB is definitely looking awesome! But a few obstacles remain if we want to give it a try.
First, its has very specific hardware requirements needing- a dedicated  Quad-core machine with at least 4GB of memory and a 2.6Gz Processor 

对硬件要求更高
Secondly, The latest Beta version only runs on Ubuntu  10.10 64Bit (which is kind of strange that they would only test on  deprecated software...) The stable version can also run on 32Bit but  otherwise has the same requirements.

最新版本只认Ubuntu 10.10 64Bit操作系统
It can run on a VM but performance suffers considerably.可以以虚拟机的形式直接部署，但是性能可想而知。
If we could find someone with an old Quad Core desktop they're getting rid of, it might be fun to set up our own remote server somewhere with a  fast internet connection? Perhaps even host our own LocalWiki or  Wordpress off of it?

 

https://moodle.org/mod/forum/discuss.php?d=227986

Anyway, here are the areas I'll compare them:

**1- Chat module:**

In bbb chat module is vertically placed on the right by defaults,  which give you the opportunity to view nearly all the messages from the  participants. In om, however, it is placed horizontally in a rectangular box just beneath the whiteboard in which you can see the very few  messages from the participants even if you hide the admin panel

Prior to 0.81 version, bbb was showing certain characters of the  participants' names (such as Grah.. instead of full name Graham Stone).  But in 0.81 beta version this was corrected and full name of the  participants can be seen.

Again prior to 0.81 version of bbb, you could change the font color  of your messages. This was a very good feature especially in  discriminating the messages. However, in 0.81 version you have only one  color: black. I believe this will be corrected since 0.81 version is  still in beta

In bbb there is also a private message area in which you can send  private messages to the users similar to adobe connect. I say similar  because in adobe connect the teacher is informed if a participant sends a private message to the other. In bbb, teacher is not informed if any of the users are privately chatting unless the recipient is the teacher  himself. In om, afaik there is not a private chat feature available.

**2- Users block:**

In om, users are given more permissions such as turning on their mic  even if you disable them to avoid echo. bbb is more aggressive than om  in this aspect: Once the moderator logs in and set turn off all mics,  every new comer enters the conference as their mics off. This is a good  feature to avoid echo since every user does not use a headset with  mic. And in om, if you choose webinar as conference type, the mics can  be bulk turned off. But this time, if you want to allow a participant to speak, you have to first click on the user's name then click again to  turn on his/her mic, which requires a double click to turn on and double click to turn off. This might cause problems if you have a crowded  class.

**3- Tools:**

The tools in om is undoubtedly richer than the ones in bbb;  especially the icons and type tool. In bbb 0.81 beta version type tool  is added, through which you can type on the whiteboard. However, the  typed items cannot be dragged to another place as it can be done in om.

In om you have one useful too, which is polls, through which you can  create polls (let's say questions) and ask the participants to make  their choices when they are ready. At the end of the voting session a  pie chart is shown.

**4- Uploaded materials:**

The materials uploaded by the teacher in om remains in teacher's  account even if s/he logs out the conference and backs to moodle. In  bbb, however, once the teacher logs out and logs in again, s/he has to [upload](https://moodle.org/mod/glossary/showentry.php?courseid=5&eid=19&displayformat=dictionary) the materials again to use them. 

In om, you can have more than one whiteboards whereas is in bbb you can use only one whiteboard.

**5- Recording:**

In om, you can record the session with the editing tools. I mean if  you draw something on the whiteboard, it is recorded as well. However,  in bbb the things that you draw on the whiteboard is not shown in the  recording; only the slides and the voice of the teacher. I think this is because of the format of the recording. The slides are saved as .swf  files and audio as .vaw. And they are saved separately. Another point to consider is if you have sessions one after another, you should think  about the recording process one more time. I mean while the recording of the older session takes place, another live session will tried to be  handled by the [server](https://moodle.org/mod/glossary/showentry.php?courseid=5&eid=30&displayformat=dictionary): How a powerful server you have, you should think about the server load.

In om, you can also define which areas of the screen to record. By  this way you can record let's say the video view of the teacher. In bbb, however, only the whiteboard and chat module is recorded.

As a solution, you can think about using a screen capture program.  Besides commercial ones if you have a Ubuntu machine you should give a  try to **kazaam**, which gives very good quality output with a small file size.

**6- Installation:**

You can install both moodle and om on the same server since om runs  on port 5080 while moodle runs on port 80. But personally I do not  recommend running the lms and video conferencing tool on the same  server, especially for production sites with large amount of  participants. But if you have a small number of participants and budget, this can be considered. Although there are some attempts to run bbb on a different port apart from 80, (personally I tried it too) it stopped  working after a successful 30 seconds runtime. 

BBB developer Fred Dixon is very decisive on this matter; bbb requires a dedicated server and has to be run on port 80 (see [here](https://groups.google.com/d/msg/bigbluebutton-setup/6FnmUN4mKHE/ArDyhapQMksJ)). From a technical aspect, from the outputs of our bbb server, I can say  that bbb heavily relies on the processor rather than RAM of the server. 

You can install bbb on a Ubuntu 10.04 server easily, and you might  have problems if you want to install it on a 12.04 Ubuntu server. See [here](http://code.google.com/p/bigbluebutton/wiki/InstallationUbuntu) for installation instructions.

You can install om on Ubuntu 12.04 and even on Ubuntu 12.10 server. See [here](https://cwiki.apache.org/confluence/download/attachments/27838216/Installing+OM2.x+On+Ubuntu64+-+Headless+v12.04.pdf?version=1&modificationDate=1359899670000) for installation instructions

**7- Layout:**

bbb 0.81 beta version is added a new feature through which you can  change the layout automatically with one click. The options are video  chat, meeting, webinar, lecture assistant and lecture.

You can use om more than a web conferencing tool. I mean you can  create different sessions, calendar events, user account assignments  through the om portal itself. In bbb, you do not have much choice but  just using video conferencing features.

**Conclusion:**

Since we have been using moodle  2.5 and horizontal chat module is extremely important for us (we are  using it in a language course and we frequently ask participants to  share their replies/comments with us) we went back to using bbb. One  more thing to add is that, personally I think, the speed of these  videoconferencing tools mostly depends on the bandwidth of the server  and the participants' rather than the architecture on which they are  built. Best regards