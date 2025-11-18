# HTB-Attacking_Web_Applications_with_Ffuf

## Table of Contents
1. [Basic Fuzzing](#basic-fuzzing)
    1. [Directory Fuzzing](#directory-fuzzing)
    2. [Page Fuzzing](#page-fuzzing)
    3. [Recursive Fuzzing](#recursive-fuzzing)
2. [Domain Fuzzing](#domain-fuzzing)
    1. [Sub-domain Fuzzing](#sub-domain-fuzzing)
    2. [Filtering Results](#filtering-results)
3. [Parameter Fuzzing](#parameter-fuzzing)
    1. [Parameter Fuzzing - GET](#parameter-fuzzing---get)
    2. [Value Fuzzing](#value-fuzzing)
4. [Skill Assessment](#skills-assessment)

## Basic Fuzzing
### Directory Fuzzing
#### Tools
1. ffuf
#### Challenges
1. In addition to the directory we found above, there is another directory that can be found. What is it?

    We can solve this by using `ffuf` with `-fs 986` flag to hide any result that have size 986. By this, we can avoid noise.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://83.136.254.49:43577/FUZZ -t 200 -fs 986
    ```
    The answer is `forum`.

### Page Fuzzing
#### Challenges
1. Try to use what you learned in this section to fuzz the '/blog' directory and find all pages. One of them should contain a flag. What is the flag?

    We can solve this by usng `ffuf` again with `-fs 0` to avoid noise.
    
    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://83.136.254.49:43577/blog/FUZZ.php -t 200 -fs 0
    ```
    ![alt text](<Assets/Page Fuzzing - 1.png>)

    We can visit `/home.php`. The answer is `HTB{bru73_f0r_c0mm0n_p455w0rd5}`.

### Recursive Fuzzing
#### Challenges
1. Try to repeat what you learned so far to find more files/directories. One of them should give you a flag. What is the content of the flag?

    We can use `ffuf` with this command to get the flag location.

    ```bash
    ffuf -ic -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://83.136.254.49:43577/FUZZ -recursion -recursion-depth 1 -e .php -t 2000 -fs 986
    ```
    ![alt text](<Assets/Recursive Fuzzing - 1.png>)

    We will find the correct path is `83.136.254.49:43577/forum/flag.php`. The answer is `HTB{fuzz1n6_7h3_w3b!}`.

## Domain Fuzzing
### Sub-domain Fuzzing
#### Challenges
1. Try running a sub-domain fuzzing test on 'inlanefreight.com' to find a customer sub-domain portal. What is the full domain of it?

    We can `fuzz` by using `ffuf` again. But instead of fuzzing folder, we will be fuzzing sub-domain.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/
    ```
    ![alt text](<Assets/Sub-domain Fuzzing - 1.png>)

    We will find the correct sub-domain is `customer`. The answer is `customer.inlanefreight.com`.

### Filtering Results
#### Challenges
1. Try running a VHost fuzzing scan on 'academy.htb', and see what other VHosts you get. What other VHosts did you get?

    Firs, we need to edit `/etc/hosts`. So if we tried to access `academy.htb` that it will knows to go to the correct ip.

    ```bash
    sudo sh -c 'echo "83.136.254.49  academy.htb" >> /etc/hosts'
    ```
    Then, we can run ffuf.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:43577/ -H 'Host: FUZZ.academy.htb' -fs 986
    ```
    ![alt text](<Assets/Filtering Results - 1.png>)

    The answer is `test.academy.htb`.

## Parameter Fuzzing
### Parameter Fuzzing - GET
#### Challenges
1. Using what you learned in this section, run a parameter fuzzing scan on this page. What is the parameter accepted by this webpage?

    We need to update `/etc/hosts`, adding `admin.academy.htb`. Then we can run `ffuf`.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:43577/admin/admin.php?FUZZ=key -fs 798
    ```
    The answer is `user`.

### Value Fuzzing
#### Challenges
1. Try to create the 'ids.txt' wordlist, identify the accepted value with a fuzzing scan, and then use it in a 'POST' request with 'curl' to collect the flag. What is the content of the flag?

    To solve this, first we need create ids.txt.

    ```bash
    for i in $(seq 1 1000); do echo $i >> ids.txt; done
    ```
    Then we can use `ffuf` to do fuzzing.

    ```bash
    ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:43577/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 768
    ```
    ![alt text](<Assets/Value Fuzzing - 1.png>)

    We can see the correct value is `73`. Then we can use curl to retive the flag.

    ```bash
    curl http://admin.academy.htb:43577/admin/admin.php -X POST -d 'id=73' -H 'Content-Type: application/x-www-form-urlencoded'
    ```
    The answer is `HTB{p4r4m373r_fuzz1n6_15_k3y!}`.

## Skills Assessment
1. Run a sub-domain/vhost fuzzing scan on '*.academy.htb' for the IP shown above. What are all the sub-domains you can identify? (Only write the sub-domain name)

    First we need to edit `/etc/hosts` file.

    ```bash
    sudo sh -c 'echo "83.136.251.210  academy.htb" >> /etc/hosts'
    ```
    Then we can run `ffuf` to find all vhosts.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:43016/ -H 'Host: FUZZ.academy.htb' -fs 985
    ```
    The answer is `test, archive, faculty`.

2. Before you run your page fuzzing scan, you should first run an extension fuzzing scan. What are the different extensions accepted by the domains?

    First we need to edit `/etc/hosts` again to be like this.

    ```bash
    83.136.251.210  academy.htb test.academy.htb archive.academy.htb faculty.academy.htb
    ```

   Then we can try to fuzz the extension.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://faculty.academy.htb:43016/indexFUZZ
    ```
    ![alt text](<Assets/Skills Assessment - 1.png>)

    We can see 3 results. The asnwer is `.php, .phps, php7`.

3. One of the pages you will identify should say 'You don't have access!'. What is the full page URL?

    After doing trial and error,  I found that `faculty.academy.htb` has `/courses` folder. Here the command that i used to find that.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://faculty.academy.htb:43016/FUZZ -ic -t 2000
    ```
    Then, i tried to fuzz *.php, *.phps, and *.php7. I found interesting file in the *.php7 result.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://faculty.academy.htb:43016/courses/FUZZ.php7
    ```
    ![alt text](<Assets/Skills Assessment - 2.png>)

    When we check `http://faculty.academy.htb:PORT/courses/linux-security.php7`, we got this.

    ![alt text](<Assets/Skills Assessment - 3.png>)

    So the correct answer is `http://faculty.academy.htb:PORT/courses/linux-security.php7`.

4. In the page from the previous question, you should be able to find multiple parameters that are accepted by the page. What are they?

    To solve this, first we can try to fuzz the GET parameter.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:43016/courses/linux-security.php7?FUZZ=key 
    ```

    ![alt text](<Assets/Skills Assessment - 4.png>)

    We got `user` as the correct parameter. Then we can try to fuzz the POST parameter.

    ```bash
    ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://faculty.academy.htb:39871/courses/linux-security.php7 -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs 774
    ```

    ![alt text](<Assets/Skills Assessment - 5.png>)

    We got `user` and `username` as the result. We can combine both result to get the answer. The answer is `user, username`.

5. Try fuzzing the parameters you identified for working values. One of them should return a flag. What is the content of the flag?

    In the previous, we have found the correct parameter. Now we need to find the correct value of it. We can use `/opt/useful/seclists/Usernames/Names/names.txt` as a wordlist.

    ```bash
    ffuf -w /opt/useful/seclists/Usernames/Names/names.txt:FUZZ -u http://faculty.academy.htb:39871/courses/linux-security.php7 -X POST -d 'username=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 774
    ```

    ![alt text](<Assets/Skills Assessment - 6.png>)

    We can see, `harry` is the valid value. Then we can try to curl with that value.

    ```bash
    curl http://faculty.academy.htb:39871/courses/linux-security.php7 -X POST -d 'username=harry' -H 'Content-Type: application/x-www-form-urlencoded'
    ```
    The answer is `HTB{w3b_fuzz1n6_m4573r}`.

