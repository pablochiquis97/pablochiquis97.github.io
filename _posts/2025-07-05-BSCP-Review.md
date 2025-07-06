---
title: "My Experience with BSCP: Analysis, Tips, and Tricks"
date: 2025-07-06 14:10:00 +0800
author: pablo
pin: true
categories: [certifications, BSCP]
tags: [
    BSCP
    Review
    Pentesting
    AppSec
  ]
image:
  path: /assets/img/BSCPExam/BSCPCertificate.png
---

Hello! If you're reading this, it's very likely that you're interested in taking the PortSwigger BSCP exam. In this post, I want to share my personal experience with the exam, how I prepared, what study materials I recommend, and some key tips that can help you pass it successfully.

---

## About me & **Background**

Ever since I discovered the world of cybersecurity during the pandemic, I've always been drawn to its offensive side, especially topics like pentesting and red teaming. For this reason, I focused on learning and strengthening my skills as a pentester, which eventually led me to take and obtain certifications like eCCPT and CAPEN. However, I felt I needed to reinforce my knowledge in web hacking and AppSec, so I identified three key certifications in that area: Hack The Box's CBBH, INE's eWPT, and PortSwigger's BSCP.

At that point, I had to decide which certification best aligned with my goals. I started exploring the CBBH, as Hack The Box offers a monthly subscription of only $8 for students through its Academy platform, which seemed like an excellent option. I took advantage of this opportunity to complete the *Bug Bounty Hunter* path, specifically designed by HTB as preparation for their CBBH certification.

![image.png](/assets/img/BSCPExam/image.png)
_CBBH Path Completed_

I thoroughly enjoyed and learned a lot from completing the CBBH path. Hack The Box's material is, honestly, very good: the way they explain concepts and the quality of the labs make it highly recommendable. However, I felt there was still a lack of deeper coverage in some key topics, and that's why it didn't quite align with my main goal: to delve as deeply as possible into everything related to web hacking and AppSec.

It was then that I finally decided on the BSCP certification. After reading several reviews, most agreed that it's a demanding certification, where many candidates need more than one attempt to pass. This makes it clear that you must have concepts very well assimilated and a solid foundation in web hacking to overcome it. In addition, its affordable price of $99 finally convinced me.

As for INE's eWPT, many opinions mentioned that its content was somewhat basic, and since my focus was to learn as much as possible about web security, I decided to leave it aside for the moment. Perhaps in the future, I will consider the new version, eWPTXv3, which, according to several comments, is much more challenging and in-depth.

Sorry for the long introduction, but I think it was important to share all this context and my thoughts before fully diving into the certification review.

## **What is BSCP?**

The **BSCP** is an official certification granted by the creators of Burp Suite, aimed at web security professionals. It demonstrates a deep understanding of web vulnerabilities, the right mindset for ethically exploiting them, and advanced skills in using Burp Suite for penetration testing.

📚 **The certification deeply covers topics such as:**

- SQL Injection
- Cross-Site Scripting (XSS)
- Access control vulnerabilities (horizontal and vertical)
- Server-Side Request Forgery (SSRF)
- Business logic vulnerabilities
- Web shell exploitation
- Advanced use of tools like Repeater, Intruder, Decoder, Comparer, and extensions

...and many more.

🧪 The exam lasts 4 hours, costs $99 USD, and is divided into three levels:

- **Level 1:** Access standard user accounts
- **Level 2:** Escalate privileges to gain administrative access
- **Level 3:** Exploit the administrative interface to read a local file

It is important to note that to take the exam, you need an active version of Burp Suite Professional, as upon successful completion, you will need to upload your Burp project to the exam platform. PortSwigger offers the possibility of obtaining a 30-day free trial if you have an educational email, which was the option I used in my case. So, if you don't have an active license, this is a totally valid alternative.

Another important aspect to consider during the exam is that it is a **proctored** test. Before starting, the platform will ask you to upload an image of your ID, activate your camera, and share your screen. Although some blogs mention that, once the identification process is complete, it is possible to close the proctoring tab, I preferred to leave it open to avoid potential issues.

I remind you that the exam is **open-book**, so you can use your notes and automated tools, such as **sqlmap**, if you deem it necessary.

## Web Security Academy

PortSwigger's Web Security Academy is, without a doubt, the best place to prepare for the exam, and this is due to several reasons. First, the exam environment is practically identical to that of the academy labs, so if you are already familiar with them, you will feel comfortable from the first moment. Second, some exam vulnerabilities might be familiar if you have previously worked on similar labs. Of course, even if they repeat, you will probably have to adapt the approach or modify the exploit according to the context of the environment. Third, all vulnerabilities that may appear on the exam are covered in the academy, which means its content is more than sufficient to pass successfully.

PortSwigger does not specify exactly which modules you should complete before taking the exam; it simply recommends having completed all "Apprentice" and "Practitioner" level labs. In my case, I also completed some "Expert" level labs to further strengthen my preparation. If it serves as a reference, this was my progress in the academy at the time of taking the exam:

![image.png](/assets/img/BSCPExam/image%201.png)

PortSwigger recommends placing special emphasis on certain labs, so I suggest you review them very carefully and keep them in mind when taking the exam.

![image.png](/assets/img/BSCPExam/image%202.png)

## About the exam

As I mentioned previously, the exam lasts 4 hours, costs $99 USD, and is divided into two web applications, each with three levels:

- **Level 1:** Access standard user accounts
- **Level 2:** Escalate privileges to gain administrative access
- **Level 3:** Exploit the administrative interface to read a local file

There are no mandatory requirements to take the exam, but it is essential to be well prepared. Therefore, I recommend carefully reading the following guides, which clarify all important aspects of the exam in detail:

- [https://portswigger.net/web-security/certification/practice-exam](https://portswigger.net/web-security/certification/practice-exam)
- [https://portswigger.net/web-security/certification/how-it-works](https://portswigger.net/web-security/certification/how-it-works)
- [https://portswigger.net/web-security/certification/practice-exam](https://portswigger.net/web-security/certification/practice-exam)
- [https://portswigger.net/web-security/certification/exam-hints-and-guidance](https://portswigger.net/web-security/certification/exam-hints-and-guidance)

To approach the exam in the best way, I recommend focusing your analysis according to the access level you are in within the application. For example, it would not make sense to look for vulnerabilities like `OS Command Injection` if you have not yet gained administrator user access.

To help you with this approach, I share the following checklists, organized by access level and focused on the most probable vulnerabilities at each stage of the exam. (In the resources section, you will find references to where I obtained these images).

![image.png](/assets/img/BSCPExam/image%203.png)

![image.png](/assets/img/BSCPExam/image%204.png)

Interactive checklist:

[https://bscp.guide/](https://bscp.guide/)

## How to prepare?

I believe that preparation time and the approach to it can vary greatly depending on each person's background and prior experience in web hacking. Therefore, I want to share my preparation process, along with some recommendations and adjustments I would make if I had to take the exam again, now that I know its structure and level of demand.

- First, I recommend completing all labs on all topics at the "Apprentice" and "Practitioner" levels.
- Once you have completed all the labs, I recommend doing them again, but this time taking detailed notes on the methodology used in each one. This will allow you to create your own personalized *cheat sheet*. Although there are many ready-made *cheat sheets* on the internet, developing your own will help you better internalize the concepts and reinforce what you have learned.
- In the following image, I show you an example of how I organized my notes using Notion, in this case about the CSRF vulnerability. Creating your own *cheat sheet* will not only be useful for passing the exam but also for being able to extrapolate all that knowledge and methodology to real-world penetration testing scenarios or bug bounty programs.

![image.png](/assets/img/BSCPExam/image%205.png)

- Two of the best *cheat sheets* I found were the following. I recommend comparing them with your own *cheat sheet* and adding anything you deem necessary to complete it.
    - [https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study)
    - [https://github.com/DingyShark/BurpSuiteCertifiedPractitioner](https://github.com/DingyShark/BurpSuiteCertifiedPractitioner)
- When working on the labs, don't limit yourself to just completing the objective presented to you. Always try to go a step further and raise the criticality of the vulnerability. For example, in XSS labs, most challenges will simply ask you to generate an alert with `alert()`. In addition to completing that step, try to think about how you could steal a user's cookie, what encoding techniques you could apply, or what other ways you could achieve the same result. This deeper approach will help you develop a stronger offensive mindset applicable to real-world scenarios.
- Once you have completed all the labs and have your *cheat sheet* ready, it's time to tackle the *Mystery Labs* offered by PortSwigger. In these challenges, you are presented with a lab without any context or indication of the vulnerability you need to find, which makes them ideal for testing your methodology and the effectiveness of your *cheat sheet*. PortSwigger recommends performing at least 5 *Mystery Labs* before taking the exam. In my case, I easily completed more than 200, but that depends on each person's pace and confidence level. The important thing is to practice them until you feel truly comfortable solving them on your own.

![image.png](/assets/img/BSCPExam/image%206.png)

- Something I didn't like about the *Mystery Labs* is that, after a certain point, they tend to repeat quite a bit and are not as random as one would wish. To solve this, I used a Python script that extracts all PortSwigger labs into an HTML file—you can find it at the following [link](https://github.com/ossamayasserr/PortSwigger-Labs-in-1-Page)—and, with the help of ChatGPT, I developed a small additional script that selects a random lab to practice more dynamically.
- At this point, you're probably almost ready to take the BSCP exam. The best way to test your knowledge, methodology, and *cheat sheet* is by solving the *Practice Exams* provided by PortSwigger. These practice exams are divided into three phases, just like the real exam, making them an excellent tool to realistically evaluate your preparation.
- Below, I share my *write-up* of the *Practice Exam*, but I recommend reading it only if you have already solved it on your own, to avoid potential *spoilers*. Taking the *Practice Exam* without help and 100% on your own is, without a doubt, the best way to gauge your current level and know if you are truly prepared to face the official exam.

## Burp Extensions

- As you progress and solve labs in the Web Security Academy, various Burp Suite extensions are suggested. Below, I share the ones I used during my preparation and on the exam:
    - [Param Miner](https://portswigger.net/bappstore/17d2949a985c4b7ca092728dba871943)
    - [**HTTP Request Smuggler**](https://portswigger.net/bappstore/aaaa60ef945341e8a450217a54a11646)
    - [**Content Type Converter**](https://portswigger.net/bappstore/db57ecbe2cb7446292a94aa6181c9278)
    - [**JWT Editor**](https://portswigger.net/bappstore/26aaa5ded2f74beea19e2ed8345a93dd)
    - [**Server-Side Prototype Pollution Scanner**](https://portswigger.net/blog/server-side-prototype-pollution-scanner)
    - [**DOM Invader**](https://portswigger.net/burp/documentation/desktop/tools/dom-invader)
    - [**Java Deserialization Scanner**](https://portswigger.net/bappstore/228336544ebe4e68824b5146dbbd93ae)
    - [**Hackvertor**](https://portswigger.net/bappstore/65033cbd2c344fbabe57ac060b5dd100)

## My Exam Experience

To be honest, I passed the certification on my second attempt. The first was somewhat disastrous: I failed to pass any of the three phases in any of the two applications. I believe this was due to several factors. Although I had completed many *Mystery Labs* and felt confident identifying vulnerabilities, I realized that I didn't have the concepts as internalized as I thought. In many cases, I solved the labs mechanically, without truly reasoning why certain techniques worked or failed. I didn't thoroughly analyze why a specific encoding made an exploit successful, and that cost me.

That first attempt was key to detecting my knowledge gaps and starting to work on them more consciously. I realized that, while I could follow the steps to solve a lab, I lacked a deeper understanding of the 'why' behind certain exploits, especially in areas like:

* **Encodings and Obfuscation:** Often, I didn't fully understand why a specific encoding technique made an exploit successful or why it failed with another. For this, in addition to the PortSwigger guide on obfuscation that I already mentioned ([https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)), I focused on practicing scenarios where input sanitization was strict, forcing myself to try various encoding combinations (URL, HTML, Unicode, etc.) until the payload passed.
* **Business Logic and 'Chain' Vulnerabilities:** I noticed that, although I could find individual vulnerabilities, I found it harder to chain them to achieve a larger objective (like privilege escalation). For this, the Web Security Academy's Mystery Labs were invaluable, but I also looked for complex Bug Bounty write-ups on HackerOne and other platforms, paying special attention to how researchers combined minor findings to achieve critical impact.
* **Advanced Use of Burp Extensions:** As I mentioned, not having my Burp Suite extensions properly configured or understanding their real scope hurt me. I spent time reading the documentation for each key extension (Param Miner, JWT Editor, etc.) and testing their functionalities in the labs to understand how to optimize their use and not rely solely on automatic scanning.

Another factor that negatively influenced me was the lack of proper preparation and configuration of my Burp Suite extensions. During my practice, I used a virtual environment with Kali Linux, but for the exam, I installed the Pro version of Burp Suite on Windows. To my surprise, several extensions didn't work as well or took much longer to detect vulnerabilities compared to Linux. This was a critical error, as it's a four-hour exam, every minute counts, and the exam is designed for you to leverage Burp and its extensions as efficiently as possible, often automating part of the work.

Fortunately, the application environment worked stably and quickly, without any technical conflicts.

### Attempt 2

For my second attempt, I had already identified the vulnerabilities where I had the biggest gaps and made sure to correct them. Thanks to that, I felt much more prepared and confident. This time I had all my tools ready and correctly configured, and I opted to use a virtualized environment with Kali Linux 2025.

My strategy was to manually audit one of the applications while letting Burp Suite's automatic scans run on the other. I quickly identified the first vulnerability, although its exploitation took me some time. Forty minutes into the exam, I had already completed Phase 1 in one of the applications. However, that's when the problems began: the lab environment started behaving erratically, the applications became extremely slow, and I could barely interact with them.

I tried changing browsers, restarting Burp Suite, and even considered it might be a problem with my connection, but upon checking that it was stable, I concluded that the problem came directly from the exam environment. In the midst of frustration, I decided to write an email to the PortSwigger team informing them of the situation, but due to the time difference, they were not during business hours.

This scenario was very demotivating, as in my first attempt the environment had worked without issues. Moreover, it coincided with other technical problems I had noticed in the platform's labs during those days, which made me suspect it was a general system instability. By then, I had already lost about an hour, which left me with only two hours to complete the rest of the exam.

Fortunately, at the beginning of the third hour, the applications started responding correctly, and I was able to continue with the audit. Although I was aware that I probably wouldn't have enough time to complete the entire exam, I decided to stay focused and not let the situation demotivate me. The privilege escalation and reading of the internal file took me about an hour. That meant I had just over an hour left to tackle the second application.

I maintained the best possible attitude, and luckily, in the second application, I quickly identified the exploitation path. Thanks to that, I managed to complete it successfully. In the end, I finished both applications with barely ten minutes remaining before the four hours of the exam ran out.

## Tips

- One of the fundamental keys to successfully completing the exam is to have a well-structured and complete *cheat sheet* that includes all vulnerabilities covered in the Academy along with their different exploitation techniques. Since the exam lasts only four hours, time can be very limited, and having this reference at hand will allow you to save valuable minutes and maintain a more efficient approach.

- Time management is, without a doubt, one of the most critical factors for success in the BSCP exam. With only 4 hours available for two applications and three phases for each, every minute counts. Based on my experience, and especially in my second attempt where time was a challenge, I recommend the following approximate time distribution per application and phase:
    * **Phase 1 (Standard account access):** Try to complete this phase in the first application in about **45-60 minutes**. Once achieved, replicate the methodology or look for the initial vulnerability in the second application. It is vital not to get stuck here, as without this access, later phases are impossible. 
    * **Phase 2 (Privilege escalation to administrator):** This phase may take a little longer. Aim for about **60-75 minutes** per application. This is where creativity and a deeper knowledge of access control vulnerabilities, business logic, or CSRF come into play.
    * **Phase 3 (Local file reading):** This is usually the most challenging phase and where "thinking outside the box" is key. Allocate about **45-60 minutes** per application. Web shells, command injections, or deserializations can be the path here.
    * A crucial tip is **don't get stuck**. If you've been on a vulnerability for more than 20-30 minutes with no clear progress, make a note of it and move on to another possible avenue. Sometimes, changing your approach or even moving to the other application can refresh your perspective. Burp Suite allows you to easily switch between projects, so take advantage of the possibility of leaving active scans on one application while you work manually on another.

- If you are sure that you are exploiting correctly but cannot get it to work, try different obfuscation techniques. Often, small changes in encoding or how the payload is sent can make a difference. Therefore, I recommend reviewing the following guide, which compiles various useful obfuscation techniques for these types of scenarios:
    - [https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings](https://portswigger.net/web-security/essential-skills/obfuscating-attacks-using-encodings)
- If you successfully complete the exam, I recommend taking screenshots of both applications as proof that you finished all phases. It's also good practice to save evidence of when you upload your Burp Suite project to the platform.
As I had detailed with my experience with the exam, in my case the environment did not behave optimally and the project was not uploaded correctly to the web, so the PortSwigger team asked me to send it via email. Thanks to having screenshots and records of the entire process, I was able to demonstrate that I had completed the exam without issues. That's why I always recommend keeping evidence of every step, as it can be key in case of any eventuality
- If you don't pass the certification, be sure to save your Burp Suite project file. This will allow you to review it calmly later and analyze the application's requests, responses, and flows in detail. That way, you can identify attack vectors that you might have overlooked during the exam and learn from your mistakes to improve on the next attempt.
- A very useful technique to check if you really master a topic is to try to teach it to someone else, or even explain it out loud as if you were doing so. By verbalizing the concepts, you will quickly realize any gaps you may have and can strengthen those areas more effectively.
- Always try to think "outside the box"; developing this skill is fundamental to becoming good security analysts. Often, the most interesting or critical vulnerabilities are not obvious, and it is this creative and lateral thinking that makes the difference in real-world scenarios.
- I strongly recommend exploring PortSwigger's research section ([https://portswigger.net/research](https://portswigger.net/research)) and watching the videos shared there. In particular, James Kettle's talk on *HTTP Request Smuggling*, presented at DEFCON, is simply magnificent. Not only is it highly didactic, but it is also very inspiring, as it demonstrates how many of the labs available in the Academy are based on real scenarios discovered during the PortSwigger team's investigations.
- I believe it is always essential to maintain a *growth mindset*—not just for this certification, but for any challenge in general—especially in a field like cybersecurity, where everything evolves rapidly. Stay open to constant learning, accept mistakes as part of the process, and, above all, try to enjoy every stage of the journey. In the end, the most valuable thing is not just the certification, but everything you learn along the way.

## Conclusion

The BSCP certification was a demanding yet extremely enriching challenge. I enjoyed every stage of the process, from preparation to obtaining the certificate. Along the way, I acquired deep knowledge of web hacking and AppSec, so I would undoubtedly recommend it to anyone wishing to develop in these areas.

As with any challenge, I faced obstacles and moments of frustration, but I am convinced that the most important thing is to maintain a positive attitude, have a growth mindset, and enjoy the learning process.

### Practical Application and Future

Now that I have obtained the BSCP certification, my main goal is to apply all this knowledge practically. It's not just about having a certificate, but about the ability to identify and exploit vulnerabilities in real-world scenarios.

* **In Pentesting:** The methodology and depth in using Burp Suite learned during BSCP preparation are directly transferable to my web penetration testing projects. I feel much more confident in approaching complex applications, identifying less obvious attack vectors, and developing more sophisticated exploits.
* **In Bug Bounty Programs:** This certification has strengthened my 'eye' for finding vulnerabilities and my ability to report them effectively, increasing my chances of success on platforms like HackerOne or Bugcrowd. The 'going a step further' mindset I developed while solving the labs is fundamental for discovering unique findings.
* **Continuous Professional Development:** The field of web cybersecurity is constantly evolving. The BSCP has provided me with a solid foundation to continue learning about new attack and defense techniques, keeping me relevant in this area. My next step is to continue exploring more advanced vulnerabilities and contribute to the community by sharing my findings and knowledge.

I hope you enjoyed this review and found it useful in your preparation for the certification. If you wish to contact me or have any questions, my social media channels are available.

Happy Hacking! 🔐💻

## Resources

### Cheat Sheet:

- [https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study](https://github.com/botesjuan/Burp-Suite-Certified-Practitioner-Exam-Study)
- [https://github.com/DingyShark/BurpSuiteCertifiedPractitioner](https://github.com/DingyShark/BurpSuiteCertifiedPractitioner)
- [https://bscp.guide/](https://bscp.guide/)

### Youtube / Courses:

- S4vitar & Hack4u: In my opinion, they offer the best content in Spanish. S4vitar always tries to go a step further when facing a machine or lab, which is reflected in the depth of his approach. His new course *Hacking Web*, available in his academy, where he solves all PortSwigger labs, is an excellent tool to prepare specifically for this certification.
    - [https://www.youtube.com/@S4viSinFiltro](https://www.youtube.com/@S4viSinFiltro)
    - [https://hack4u.io/](https://hack4u.io/)
- Gonzalo Carrasco: Excellent for learning about OAuth vulnerabilities.
    - [https://www.youtube.com/@gonzalocarrasco](https://www.youtube.com/@gonzalocarrasco)
- Rana Khalil: Explains excellently and very didactically, which is perfect for improving your understanding and learning how to apply an effective methodology against different classes of vulnerabilities.
    - [https://www.youtube.com/@RanaKhalil101](https://www.youtube.com/@RanaKhalil101)
    - [https://academy.ranakhalil.com/](https://academy.ranakhalil.com/)
- Jarno Timmermans: Excellent for learning about HTTP Request Smuggling.
    - [https://www.youtube.com/@netletic](https://www.youtube.com/@netletic)
- Seven Seas Security: Excellent for learning about SSTI and XXE.
    - [https://www.youtube.com/@7SeasSecurity/playlists](https://www.youtube.com/@7SeasSecurity/playlists)

### Other resources:

- [https://micahvandeusen.com/blog/burp-suite-certified-practitioner-exam-review/](https://micahvandeusen.com/blog/burp-suite-certified-practitioner-exam-review/)
- [https://bscpcheatsheet.gitbook.io/exam](https://bscpcheatsheet.gitbook.io/exam)
- [https://deephacking.tech/burp-suite-certified-practicioner-review-bscp/](https://deephacking.tech/burp-suite-certified-practicioner-review-bscp/)
- [https://github.com/ricardojoserf/Portswigger-Labs](https://github.com/ricardojoserf/Portswigger-Labs)
- [https://h0j3n.github.io/posts/Burpsuite-Practice-Exam/#walkthrough-33](https://h0j3n.github.io/posts/Burpsuite-Practice-Exam/#walkthrough-33)
- [https://medium.com/@ossamayasserr/how-to-crush-bscp-exam-in-75-mins-bscp-review-0b207a17e26d](https://medium.com/@ossamayasserr/how-to-crush-bscp-exam-in-75-mins-bscp-review-0b207a17e26d)
- [https://github.com/ossamayasserr/BSCP-Notes](https://github.com/ossamayasserr/BSCP-Notes)
- [https://www.youtube.com/watch?v=w-eJM2Pc0KI&ab_channel=DEFCONConference](https://www.youtube.com/watch?v=w-eJM2Pc0KI&ab_channel=DEFCONConference)
- [https://portswigger.net/research/http2](https://portswigger.net/research/http2)