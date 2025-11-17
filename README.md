# HTB-Attacking_Web_Applications_with_Ffuf

## Table of Contents
1. [Basic Fuzzing](#basic-fuzzing)
    1. [Directory Fuzzing](#directory-fuzzing)
    2. [Page Fuzzing](#page-fuzzing)
    3. [Recursive Fuzzing](#recursive-fuzzing)
2. [Domain Fuzzing](#domain-fuzzing)
    1. [Sub-domain Fuzzing](#sub-domain-fuzzing)
    2. [Filtering Results](#filtering-results)

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