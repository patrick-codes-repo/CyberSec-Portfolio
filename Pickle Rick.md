# Scenario
This is a beginner friendly black-box exercise. The goal is to exploit a web server and uncover three secret ingredients (flags).

More details on this challenge can be found on [TryHackMe](https://tryhackme.com/r/room/picklerick).

# Tools Used
- Nmap
- Gobuster
- Netcat

# Execution
1. Visit the target web application.

![Home Page](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/home%20page.PNG?raw=true)

2. Not much useful here, so I inspect the page source code.

![Page Source](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/page%20source.PNG?raw=true)

There is a username saved here, this will likely be useful.

3. Perform an Nmap scan to check for any additional running services.

![Nmap](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/nmap.PNG?raw=true)

4. ssh is running on the server as well, I may be able to log in using the username found in the source code.

![ssh](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/ssh%20attempt.PNG?raw=true)

No luck.

5. Time to look elsewhere. I start by performing a Gobuster scan to find hidden directories.

![Directory Scan](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/directory%20scan.PNG?raw=true)

6. The /assets folder is accessible in the browser.

![Assets](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/assets.PNG?raw=true)

After investigating these files none seemed to be particularly interesting.

7. robots.txt was also discovered by the Gobuster scan and it contains unusual content. However, it does not seem particularly useful.

![Robots](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/robots.PNG?raw=true)

8. Perform another Gobuster scan, but this time, search for some common file types.

![File Scan](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/file%20scan.PNG?raw=true)

This found many more files than the previous scan.

9.  login.php is the most promising file so I will check it out first.

![Login](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/login.PNG?raw=true)

It is a login portal.

10. I already have a username, maybe the text from robots.txt will work as a password.

![Commond Panel](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/command%20panel.PNG?raw=true)

Success! I now have a command prompt.

11. I start by running ls to see what files are in my current directory.

![ls](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/ls.PNG?raw=true)

The first ingredient is here, as well as a clue.

12. I try to cat the ingredient file, but this command is blocked.

![Cat](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/cat.PNG?raw=true)

I attempt to circumvent the rule by using tail and head, but these commands were also blocked.

13. Luckily, I was able to open the first ingredient file and clue.txt in the browser.

![First Ingredient](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/ingredient1.PNG?raw=true)

![Clue](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/clue.PNG?raw=true)

14. After looking around the file system as the clue suggests I was able to find the second ingredient in the /home/rick directory. It also turns out rev is not blocked by the panel so I can use it to print the ingredient.

![Second Ingredient](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/ingredient2.PNG?raw=true)

![Jerry](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/jerry%20tear.PNG?raw=true)

This output is in reverse, so the second ingredient is actually 1 jerry tear.

15. I still do not know where the third ingredient is. To start my search I inspect the source code of the command panel and found what appears to be a base64 encoded string.

![base64](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/base64.PNG?raw=true)

After decoding this string for multiple rounds it gave me the text “rabbit hole” suggesting that this is a dead end.

16. From here I successfully created the file /home/rick/.ssh/config with the hopes of overwriting the current ssh configuration. However, when I tried to write to this file with echo, the command was blocked. The command panel is relatively prohibitive and inconvinient so I want to avoid using if possible. To do this I decided to attempt to create a reverse shell.

17. Start a listener on my machine to accept a connection.

![netcat](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/netcat.PNG?raw=true)

18. My first attempt was a bash reverse shell, this had no effect. I know the server can run PHP so I tried a PHP reverse shell next.

![Shell Code](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/shell%20code.PNG?raw=true)

The connection was successful!

![Connection](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/connection.PNG?raw=true)

19. From here I searched through the file system and did not find anything useful. The only interesting directory I found, but could not access, was /root. So my next step was to try and login as root.

![root](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/root.PNG?raw=true)

It worked!

20. I found the final ingredient in the /root directory.

![Third Ingredient](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Pickle%20Rick/ingredient3.PNG?raw=true)

# Lessons learned
1. Always scan for hidden files with common extensions. I performed a directory scan, but needed a hint to know to scan for files with extensions. I will need to get in the habit of doing this as these files may contain valuable information.

2. Basic server enumeration and exploitation techniques.
