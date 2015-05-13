---
layout: post
title:  "Tweet it loud – When under attack, let the world know"
categories: jekyll update
---

<div class="content" style="margin-top: 15px;">
        <p>Soon after I set up my web server a couple of weeks ago, attacks started coming in. It took no time to
          realize that there is little that can be done. There attacks are more or less harmless as they try to brute
          force the username and password for various services on the server. In my case, the services are sshd and FTP
          server (vsftpd). I said that the attacks are mostly harmless because it is difficult to brute force any
          decent password, more so when you don’t know the username.</p>
        <p>I see so many attacks on my SSH server trying to brute force the password for usernames ‘admin’ and ‘root’.
          Clever sshd just lets them keep guessing without telling them that the username doesn’t exist. It just sits
          there and enjoys the conversation. Actually it is fun to see the usernames/password that are tried. Just fire
          up wireshark/tcpdump when under attack and look at the packets, the username/passwords are as amusing as they
          can get. Sometimes the passwords are pure dictionary attack. Sometimes they even have a local tinge in them.
          For example, Chinese names !!!</p>
        <p>Most of the attacks are from bots. Bots running on ‘owned’ machines – compromised machines at government
          offices, educational institutes and public kiosks. Basically, this is generally how botnets work – a bot
          infects your computer, it then starts launching attacks on other computer in its quest to multiply. In
          addition it also opens up other services on the infected computer to achieve its other primary objectives.
          Remember, bots and botnets are all about money today. It is no longer a fame game. So typically a owned
          machine will have port 80 (http), 21(ftp), 22(ssh). You will also see compromised computers hosting a website
          (often porn) and running services like webmail, phpmyadmin etc.</p>
        <p>Last weekend, I thought – is there something I can do when under attack… Ummm well maybe I can become a good
          netizen and let the world know about the attack. Why would that help anyone? Well just in case someone else in
          world is getting attacked by the same IP address and chooses to type its IP address on google, he can have a
          little more information about the attack and his fellow sufferers.</p>
        <p>Okay, so I want to make the attack information quickly available to the world. So probably I can have a
          webpage of such attack information and I’ll keep updating it as and when I get an attack. Well … sort of good
          idea but not so slick. If google indexes my webpage every 30 days, it will take 30 days (worst case) for the
          information to be available on google.</p>
        <p>A better approach is to ‘tweet’ about the attack :).
          Google indexes twitter accounts far more frequently than it indexes an average website. So an twitter update
          can be indexed even within minutes after it has been tweet’ed.</p>
        <p>We* wrote a quick and dirty python script to help us do exactly that. The scripts monitors the log files
          every few minutes (15 minutes in our case). The log files are /var/log/auth.log and /var/log/vsftpd.log.<br>
          The scripts maintains an in-memory hash table of &lt;ip, [list of usernames used in attack]&gt;. Here ip is
          IP address of the attacker and it serves as an key in the hash table. The corresponding value is a list of
          all the usernames that the attacking computer tried.</p>
        <p>So the in memory hash table grows with each attack and would typically look like:<br>
          &lt; 210.90.74.163, [admin, test, root, nagios…]&gt;</p>
        <p>The following algorithm is repeated every 15 minutes:</p>
        <ul>
          <li>We check the new attacks in past 15 minutes and update our in memory hash table</li>
          <li>For all those entries that have surpassed a threshold number of tries in this timeslot are eligible to get
            tweeted. For eg. With a threshold of 5 tries, all those entries that now have more than 5 entries in the
            username list (and were not tweeted before) are deemed as attacks and are tweeted.</li>
        </ul>
          A simple twitter API for python makes tweeting as simple as possible.
          A new tweet appears on <a href="http://twitter.com/compromised_sys">http://twitter.com/compromised_sys</a>
            account.

        <p>A diagrammatic representation of the process is below:</p>
        <div class="wp-caption aligncenter" style="width: 610px">
          <img title="Overview of the entire process" src=<%=image_path "blog/compromised_sys_diagram.png"%> alt="" width="600" height="300">
          <p class="wp-caption-text">Overview of the entire process</p></div>
        <p>In case you are interested in seeing the python implementation, you can download the file below. As I said
          earlier, it is quick and dirty implementation and requires code cleanup.<br>
          <a class="wp-caption" href="https://gist.github.com/adityab4u/d32210cb43e6bc08983b">tweet_attacks.py</a><br>
          <strong><br>
            Results:</strong><br>
          The script runs in production and tweets the twitter account <a href="http://twitter.com/compromised_sys">
          http://twitter.com/compromised_sys</a> regularly. Once we had the first version up and running, we could see
          tweets getting indexed in as less as 15 minutes after updates to twitter account.</p>
        <p>*This code was co-authored by
          <a class="wpGallery" title="Aditya Mahendrakar's personal webpage" href="http://www.andrew.cmu.edu/user/amahendr/home.html" target="_blank">Aditya Mahendrakar</a></p>

  </div> <!-- content div -->
