# Scenario
This is a black-box file upload challenge. The only known is that the server, jewel.uploadvulns.thm, is vulnerable to a file upload exploit. The goal of this challenge is to discover the upload vulnerability and exploit it to get a shell and print the flag saved in /var/www/.

More details on this challenge can be found on [TryHackMe](https://tryhackme.com/r/room/uploadvulns).

# Tools Used
- BurpSuite
- Gobuster
- Hexedit
- Linux command line

# Execution
1. Test uploading a regular jpg file.

![JPG Upload](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/Innocent%20jpg.PNG?raw=true)

![JPG Upload Success](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/innocent%20upload.PNG?raw=true)

The file uploads successfully which means this jpg files can be uploaded to this site.

2. Create and upload the empty file test.jpg to check for magic number filtering. Have Burp intercept on to check if the file is ever sent to the server.

![touch test](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/create%20blank%20jpg.PNG?raw=true)

![Invalid Format Error](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/invalid%20file%20format.PNG?raw=true)

The file upload results in an “Invalid File Format” error. In addition to this Burp did not intercept any data which means the upload was caught by a client side filter. This is likely due to magic numbers or file content since the file extension is already known to be allowed.

3. Use hexedit to add the magic numbers FF D8 FF E0 to test.jpg and upload the file.

![Magic Numbers](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/magic%20numbers%20success.PNG?raw=true)

test.jpg now uploads successfully. After changing the extension of test.jpg to js the upload is caught client side for the same error seen before, “Invalid File Format”. This means that there are client side filters checking for both the file extension and magic numbers.

4. Search for where the filtering is done by inspecting the page source.

![Page Source](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/page%20source.PNG?raw=true)

There does not appear to be any filtering in the page source code. It is possible that one of the included js files performs the filtering. This will likely be done by upload.js considering the name.

5. Clear browser data and refresh the page to intercept external js files with Burp before they are loaded in the browser. Burp’s intercept rules must also be edited to include js files. upload.js is of particular interest as it seems to be the most likely to be performing client side filtering.

Delete the highlighted rule from the Burp proxy to capture incoming js files.

![Burp Rule](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/js%20rule.PNG?raw=true)

Forward messages until upload.js is intercepted.

![upload.js](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/uploadjs.PNG?raw=true)

Here it can be seen that upload.js checks for file size, file extension, and magic numbers. These checks should be removed before forwarding the response to the browser.

6. Now that the client side filters are out of the way, it is time to learn about the server. Burp intercept can be used to determine what backend language the server is running.

![x-powered-by](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/x-powered-by.PNG?raw=true)

X-Powered-By has a value of Express. A quick search for Expess.js reveals that it is a Node.js framework.

7. This means the uploaded shell will need to run on Node.js. In order to upload a shell I must first determine what the server side filter will allow through. This will be done through extensive trial and error.
The following are some of the details learned:
  - Uploading a file with a random extension results in an error. This means the server is using a whitelist as a filter. More testing revealed that the server only accepts jpg files, malformed and alternate extensions were also rejected.
  - Uploading a file with non-jpeg magic numbers or even no magic numbers at all is successful. This means there is no magic number filtering performed server side.
  - Changing the MIME type of the uploaded file with Burp results in an error. This means the server filters for MIME types. Further testing showed that image/jpeg was the only accepted MIME type.

8. Now that I have an idea of what the server is filtering for, I need to find out how and where my uploaded files are saved. The first step of this is to find directories on the server. This is done using a Gobuster scan.

![Directory Scan](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/directory%20scan.PNG?raw=true)

The content and Content directories are likely locations for the uploaded files to be stored. The admin directory may also have valuable information.

9. To find what files are in the content folders I can use another Gobuster scan using the provided wordlist.

![Content Scan](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/content%20scans.PNG?raw=true)
The content and Content directories appear to duplicates of each other. After opening the discovered files in the browser it was determined that these directories, and all their files, are indeed duplicates. ABH.jpg in content is the same file as AMH.jpg in Content. In addition to this HGX.jpg has been identified as the jpg file uploaded earlier. This means that the files I upload are renamed by the server and stored in the content directory.

![HGX](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/HGX.PNG?raw=true)

10. Now that I know that where to find uploaded files I will try to upload a script for a reverse shell. My earlier testing revealed that the uploaded file must have a jpg extension and MIME type of image/jpeg.

Create a jpg file containing the shell script.

![Shell Code](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/shell%20code.PNG?raw=true)

Intercept the file upload with Burp and confirm the MIME type is image/jpeg.

![MIME](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/MIME.PNG?raw=true)

11. The file upload was successful, now I need to find the uploaded file on the server. This can be done with another Gobuster scan.

![Gobuster Scan](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/gobuster.PNG?raw=true)

This scan shows a new file, IEY.jpg. After inspecting this file with curl and in the browser I have confirmed that it is my shell.

![new file](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/new%20file.PNG?raw=true)

![curl](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/curl.PNG?raw=true)

12. Now I need to find a way to run my shell. Accessing the URL of my image directly does not do the trick. However, the /admin page of the site shows a promising prompt.

![admin page](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/admin.PNG?raw=true)

It may be possible to use this prompt to run the shell by entering the proper file path into this prompt.

13. I will open a netcat listener on my machine to know if my shell has made a connection.

![netcat](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/listener.PNG?raw=true)

14. Entering the name of my shell file results in “Module does not exist”.

![Not Exist](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/not%20exist.PNG?raw=true)

15. After having no luck using the absolute path to my shell, /content/IEY.jpg, I tried a relative path to my shell, ../content/IEY.jpg, this made a connection!

![Connection](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/connection.PNG?raw=true)

16. I can now print the flag.

![Connection](https://github.com/patrick-codes-repo/CyberSec-Portfolio/blob/main/Resources/Upload%20Vulnerabilities/flag.PNG)

# Lessons learned
1. Always check /admin. I was able to successfully upload and find my shell but could not find a way to run the code by accessing it via its URL. I had to look at a challenge hint to know to check /admin. I should have checked this page from the start since it can often contain valuable information.

2. A Node.js server is not the same as JavaScript server. After finding that the server was using Express I found generic shell code for JavaScript. This shell code would not run on the server and I had to find code specifically for Node.js.

3. Various techniques to check for and bypass client and server side filters. Discovering what the server filtered for was particularly challenging and rewarding.


