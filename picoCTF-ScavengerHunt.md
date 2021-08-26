# CTF Write Up - picoCTF #161 - Scavenger Hunt

2021 Aug. 24<br>
Solve 6055 of 23539 Global Attempts<br>

---

## Project Description

There is some interesting information hidden around this site http://mercury.picoctf.net:5080/. Can you find it?

---

## Preparation

I'm just starting to get my feet wet with CTFs. This picoCTF site has challenges aimed at High School students, so the first problems should all be relatively easy to solve. This flag was only worth 50 points, and I had pretty quickly found a couple other smaller "web exploitation" flags on picoCTF, so I went into this puzzle feeling pretty confident. It took me a fair bit longer than I thought it would...

---

## Process

The first thing I did was load up the webpage. There were two buttons and when I clicked either of them, the action was instant. I suppose clicking a button and getting the flag is a little too much to ask for. So the next easiest thing is viewing the page source. Sure enough, right there on line 31:

>     <!-- Here's the first part of the flag: picoCTF{# -->

Right, scavenger hunts have multiple targets... Now to use the browser's DevTools. The Elements tab confirmed that everything on the site loaded at the same time. Nothing of interest displayed on the console. The Sources tab, however, reminded me that there was some Javascript in the HTML head. When taking a look at the 'myjs.js' source file, I found this commented out:

>     /* How can I keep Google from indexing my website? */

Something to do with web scrapers? There were a few more files listed as Sources that I wanted to go through before following that question: a couple of Google font files (after a quick glance, I decided they weren't the target), and a file named 'mycss.css'. I opened the .css and line 51 read:

>     /* CSS makes the page look nice, and yes, it also has part of the flag. Here's part 2: ######### */

Yep, this is definitely a CTF for students. <br>

Turning back to the .js hint, I knew Google had some automated HTML header scrapers. Instead of relying on (outdated?) memory from years ago, let's just find the answer online. Searching "prevent google from indexing site" and "where to apply noindex" gave me two options:<br>

A. `<meta name="robots" content"noindex">` in the HTML head<br>
B. `'x-robots-tag: 'noindex'` or `'none'` in the HTTP response header<br>

I looked in the DevTools again. I didn't see any meta tags in the HTML source, and in the Network tab, none of the response headers had references to our robot overloards.

```
  <head>
    <title>Scavenger Hunt</title>
    <link href="https://fonts.googleapis.com/css?family=Open+Sans|Roboto" rel="stylesheet">
    <link rel="stylesheet" type="text/css" href="mycss.css">
    <script type="application/javascript" src="myjs.js"></script>
  </head>

HTTP/1.1 200 OK
Content-type: text/html; charset=UTF-8

HTTP/1.1 200 OK
Content-Type: text/css
Content-Length: 768
Last-Modified: Tue, 16 Mar 2021 00:51:57 GMT

HTTP/1.1 200 OK
Content-Type: application/javascript
Content-Length: 642
Last-Modified: Tue, 16 Mar 2021 00:51:57 GMT
```

"Perhaps", I thought, "meta means it's not loaded." A couple of searches later and I came across an article on SpyFu.com tilted 'What is a Meta Tag (and how do they work in HTML)?' It was a good explanation, including details about the robots meta tag. They are definitely supposed to go in the `<head>`.
Realizing that a quick and easy test to see if `<meta>`s are visible from the loaded source file was to simply look at the source of a site that likely has them, I viewed the page source of that SpyFu article. Line 5:

>     <meta charset="utf-8" />

This meant our target site (mercury.picoctf.net:5080) just wasn't using them.
At this point I was lost. I took another look at the console and saw a bubble_compiled error. A StackOverflow discussion pointed at the Google Translate extension, so I disabled my extensions just to keep the screen clean.  
After more head scratching and scrolling through DuckDuckGo, I found a helpful article on Yoast.com titled "How to use meta robots tags...". It had a cool chart showing which tag values worked with which search engines, but even better was the section at the end about conflicting parameters and 'robots.txt' files.
That 'robots.txt' file name was extremely familiar. I had definitely heard / read about it multiple times (years ago). After a quick seach and review of the concept behind 'robots.txt', I went back to the tab where I had the target site and put 'http://mercury.picoctf.net:5080/robots.txt' in the URL bar. This loaded in plain text:

>     User-agent: *
>     Disallow: /index.html
>     # Part 3: #########
>     # I think this is an apache server... can you Access the next flag?

Apache, huh? I had heard of it, of course, but didn't know what made it unique or if it was vulnerable somehow. I thought maybe there was some easily reachable configuration file. If not, then at least researching apache configuraiton would help. So I searched for 'apache server config file' and found the apache.org documentation, which stated that the main configuration file is usually called httpd.conf. Naturally, I went to 'http://mercury.picoctf.net:5080/httpd.conf':

> Not Found

Yeah, of course that didn't work. It was at this point I realized how insecure it would be to have a configuration file so easily accessible. I kept researching Apache, wondering if I could somehow find a list of the typical files on an Apache server. Searches for phrases like'apache directory' and 'all files on apache server'.
After a bit I finally realized "Access" was capitalized with the last hint. Searching 'apache access' brought me to an apache.org How-To article titled "Apache HTTP Server Tutorial: .htaccess files". I quickly went to 'http://mercury.picoctf.net:5080/htaccess' and found:

> Not Found

After realizing my mistake, 'http://mercury.picoctf.net:5080/.htaccess' worked much better:

>     # Part 4: #########
>     # I love making websites on my Mac, I can Store a lot of information there.

There was no "}" at the end there, which meant I still wasn't done.

Why were Mac and Store both capitalized? I started researching phrases like 'apache store mac server' and 'how to setup mac apache server'. I went back to trying to find a standard list of files on a typical Apache webserver, but none of the articles I found were the plain directory tree I was looking for.
Like with .htaccess, I tried to access /apache, /index, /index.html, /index.php, /conf.d, and /.conf (along with ones I knew wouldn't work: /apache, /store, /mac, /.store.html, /.mac, /macstore, /storage).

After taking a long break, I came back and remembered that websites often Store cookies. So I went poking around the DevTools again. The Application tab had Local, Session, and Cookie storage drop-downs, but everything was blank. Empty! I reloaded the main page and the /robots.txt a number of times and still: nothing. The Memory tab was next.<br>
I took a heap snapshot and started expanding Constructors based on whether or not their names seemed like they might contain what I was looking for. I quickly realized that this was a huge undertaking and very verbose, so I used the Class filter field to narrow down my search to just Classes that started with "sto". I was looking at two CustomWrappableAdapter constructors. After expanding a couple of the sub-items, I confirmed to myself that I really had zero clue what I was looking at. <br>

> I just need to split up the text wall...

Time to search the internet again! 'Webdev storage event constructor' wasn't helpful. I convinced myself that Web-DAV and javascript and webhooks were all much more advanced than this entry-level CTF would make me learn. (Looking back, using the word 'webdev' in my search might have simply made the results too broad and overwhelming.)<br>
I went back to the target site and spent some time trying to find cached data. Related searches lead to PHP and Redis references. I also researched what .htaccess actually did and kept trying to find a way to list all the files on an Apache server from the browser. Attempts to access /.php.ini, /redis, /var/www, /ms, /ms.info, and /info were fruitless. Of couse, /404 worked as expected (with a 200 OK response)!<br>
It was at this point that I started looking for Apache vulnerabilities. All the Apache-specific stuff I found had already been patched years ago. Maybe I wasn't looking hard enough, but, remember: this is an entry-level flag. I did, however, read "13 Apache Web Server Security and Hardening Tips" at tecmint.com. Number 9 was "Turn off Server Side Includes..."<br><br>
I researched Server-Side Includes, HTML pre tags, and SSI injection for 15 minutes and decided I had made enough progress for the day. There was no timed element to this CTF and I wanted a full night's sleep before my meetings at work the next morning.<br>

:sleeping:

The following evening came and I opened my personal laptop back up. Time to look into Server Side Includes some more.
owasp.org had a great write-up on SSI injection. SSIs are directives that are placed in HTML pages, and evaluated on the server while the pages are being served. Sometimes it is possible to execute actions by gving the web server a script wrapped in HTML comment syntax. On a vulnerable website, an attacker might be able to submit the following as a variable via a text field:

>     <!--#exec cmd="ls" -->

After consuming a couple other articles and videos, I wanted to send that `'#exec cmd="ls"'` command to the target server. (I used 'ls' and not 'dir' because the server was apparently on a Mac.) Maybe that would tell me the name of some accessible "Store" related file! Since there were no input fields and the buttons on the main page didn't call back to the server to load new data, I tried to make separate API calls with the malicious script.
From the DevTools Network tab, I copied the main call "as fetch", pasted into the Console, and added the HTML comment to the request. Since the server wasn't expecting a user input and probably would not do anything with it if it got one, I did not expect this to work.

```
fetch("http://mercury.picoctf.net:5080/", {
"headers": {
"accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,_/_;q=0.8,> application/signed-exchange;v=b3;q=0.9",
"accept-language": "en-US,en;q=0.9",
"cache-control": "no-cache"<!--#exec cmd="ls" -->,
"pragma": "no-cache",
"upgrade-insecure-requests": "1"
},
"referrerPolicy": "strict-origin-when-cross-origin",
"body": null,
"method": "GET",
"mode": "cors",
"credentials": "omit"
});
```

It did not work the first time. So I kept trying. I placed the script in various parts of the request. I changed upgrade-insecure-requests to 0. I tried while fetching /.htaccess instead. Then I remembered reading that the Server Side Include would also work if it was in the respose header. I opened the web app security tool called Burp Suite and visited the target site via a proxy browser configured with API intercept functionality. I intercepted the response header and added the malicious HTML comment. Upon completion of the page load, nothing new or different happened.
Since SSIs were not used anywhere else on the website, I decided that the feature was most likely turned off. This challenge was only supposed to be a Scavenger Hunt, after all. At what point does a Scavenger Hunt become Breaking & Entering?
I took a long deep breath and looked at the last hint again.

> "# I love making websites on my Mac, I can Store a lot of information there."

I went to DuckDuckGo and searched for 'apache store file mac'. The fourth result was a Macworld.com forum post from 2003 titled "Prevent Apache from serving .DS_Store files". Store! So I smiled and went to "http://mercury.picoctf.net:5080/.ds_store".

> Not Found

Was it time to panic? Maybe almost. Or maybe the hint capitalized Store for a reason. "http://mercury.picoctf.net:5080/.DS_Store"

> Congrats! You completed the scavenger hunt. Part 5: #########}

Now all I had to do was put the five flag pieces together and submit it as "picoCTF{#####################################}" to "https://play.picoctf.org/practice/challenge/161?category=1&page=1".<br><br>
SUCCESS! That brought my picoGym Score up to 420!:sunglasses:

---

## Post-Mortem

I was able to learn a little bit about how Apache web servers work, gained extra practice utilizing Google Chrome's DevTools and reading HTTP headers, and had I some additional fun looking into SSI injections. My main takeaway from this challenge, however, is to be even more persistent when searching online for something I am pretty sure exists. The hints made it clear to me that there was some sort of 'store' file available. When searching for something, "it's always in the last place you look."
