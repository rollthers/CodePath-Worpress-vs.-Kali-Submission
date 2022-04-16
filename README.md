# CodePath-Worpress-vs.-Kali
Cole Guerra's 4/15 Submission of Week 7 and 8 Kali vs Wordpress Codepath 

## Vulnerability 1: Back to Basics 

Baby-steps was the goal with this first exploit. I didn't want to try and blow my brain up too quickly with the new possibilities granted to me, so I wanted to start with a simple XXS. When running WPScan, a bunch of XXS-related vulnerabilties were spotted. This one in particular caught my eye, as I had already attempted messing with the comments on WordPress prior: 

![image](https://user-images.githubusercontent.com/95894083/163651799-00bde5b5-0d45-4054-94ac-bd72b2d93e5b.png)

Now that we know comments are vulnerable to XXS, we can begin experimenting with seeing what feedback we can get. I created the following post to test out the possiblity of XXS on comments: 

![image](https://user-images.githubusercontent.com/95894083/163651958-0def5c38-3bb0-4244-b388-1e55744d5efd.png)

From there, I would comment on my post to see what code would affect my website. Out of the sample scripts provided to us on OWASP, the following code snippet seemed to work whenever the comment section was loaded on the post. 

![image](https://user-images.githubusercontent.com/95894083/163652120-5682d9f0-f14e-4abb-818f-8ad6e5ab06b9.png)

Submitting the comment makes it so that every time it is opened, an XXS error message pops up at the top of the screen. In it's current state, its more of a spooky message to unassuming users than something that poses an actual serious threat. However, this exploit proves a lack of sanitation of inputs in the comment field, allowing attackers to put much more nefarious scripts. This particular exploit was possible in any version prior to 4.1, and was patched later in 4.1.26.  

![XXS Proof](https://user-images.githubusercontent.com/95894083/163652621-095b06d7-13f0-42de-b196-5666f62d2de6.gif)

## Vulnerability 2: Behold! All of my pages 

The next avenue I wanted to explore was the possibility of exposing private pages as a guest or non-admin user. There are a couple of ways I'm sure you could do this, but I opted to start by targeting the URL of my page, attempting to take advantage of any IDORs. I figured this was a good place to start, as the url of an unoptimized WordPress page (like mine), is very malleable. Before logging out, I created a private page that can only be viewed if you have admin status. 

![image](https://user-images.githubusercontent.com/95894083/163657592-127ce56d-ac00-49b9-8b11-ae7230458d35.png)

Logging out, I started to mess with the url in simple ways. I first appended a ```/?``` to the website url and tried navigating through ids and pages. When this turned up blank (sometimes literally blank), I interrogated WPScan and found these links. 

![image](https://user-images.githubusercontent.com/95894083/163658661-4d75f7e2-eb20-45e0-a597-8a3cb45f9897.png)

When going through notes on vulnerabilities in earlier patches, one post suggested appending
```?static=1&order=asc```
When giving it a shot, these are the results. 
```http://localhost:8080/?static=1&order=asc```

![IDOR Proof](https://user-images.githubusercontent.com/95894083/163659107-70cd5042-467c-4ca9-8eea-fabaf6cc8c63.gif)

Two things of particular interest with this IDOR. One is that it not only exposed every page, both private and public, but it also revived pages that I had recently deleted. SQL Land, for example, was a page I had deleted the night before, and isn't even showing up on my admin account. The second interesting thing is that this exploit on the WPScan website was claimed to be fixed in versions 3.8 and above, yet I was still able to replicate it on my own website. 

## Vulnerability 3: Ruining my Mangum Opus 
For this last exploit, I wanted to get my hands on and use metasploit to deface a bit of my website. I wanted to give the owner (myself) a big surprise when loading up my website. 

Metasploit's entrance through the Reflex Gallery exploit puts the user in the directory: 

```/var/www/html/wp-content/uploads/2022/04```

To navigate out of this, I used a series of ```cd .. ``` commands until I was able to get to the html branch where a majority of the important directories are. 

![image](https://user-images.githubusercontent.com/95894083/163663642-2d8abcf5-3a51-47e7-bf89-1af72b8973a8.png)

With these directories visible, I started cracking into php files that affect how the website loads. The file of importance here is index.php. Using the edit command, we can take a peek inside of the file and edit it with vim (unfortunately). I decided to add an image inside of the code. This took a couple of tries, as I am rusty with php syntax, and I also did not have any reliable way to paste the image url, so I had to hand type it (yuck). Eventually the working code I got looked like this: 

```
echo '
<html>
<img src="https://img.thedailybeast.com/image/upload/v1531451526/180712-Weill--The-Creator-of-Pepe-hero_uionjj.jpg" />
</html> 
';
```
![image](https://user-images.githubusercontent.com/95894083/163663786-015cfc67-6d1b-4c66-a5ed-d495f8ef0d0f.png)

Really any image here could've worked, and I am also sure more devious code could've been input to harm the user. It isn't very hard to break the website through these files, as one stray "<" causes an error to be put up every time the website is loaded. Regardless, this is how it looks when you log into the website. 

![MS Gif 2](https://user-images.githubusercontent.com/95894083/163663992-a93c6a96-cd80-46c0-a0ca-4c14de123d6a.gif)

This exploit, and all other ones associated with the Reflex Gallery reverse shell exploit would be patched in later versions of the plugin (3.1.4). 
(As an added surprise for myself later, I also edited some css files and replaced the font with comic sans).  
