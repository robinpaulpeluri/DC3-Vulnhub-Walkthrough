# DC3 Vulnhub Walkthrough

#### Description

DC-3 is another purposely built vulnerable lab with the intent of gaining experience in the world of penetration testing.

As with the previous DC releases, this one is designed with beginners in mind, although this time around, there is only one flag, one entry point and no clues at all.

Linux skills and familiarity with the Linux command line are a must, as is some experience with basic penetration testing tools.

For those with experience doing CTF and Boot2Root challenges, this probably won't take you long at all (in fact, it could take you less than 20 minutes easily).

If that's the case, and if you want it to be a bit more of a challenge, you can always redo the challenge and explore other ways of gaining root and obtaining the flag.



## Scanning 

  ### Reconnaissance 
  
Sudo netdiscover
  
  ![Screenshot from 2023-01-17 01-14-16](https://user-images.githubusercontent.com/108471951/213867465-39a9e0fd-e30b-4742-8c6c-4a6cefbe7492.png)

Firstly we have to identify the machine IP address using tool like NetDiscover

### Nmap

nmap -sC -sV -p- 192.168.0.106


![Screenshot from 2023-01-21 18-20-04](https://user-images.githubusercontent.com/108471951/213867561-351b64f7-3804-4da4-90db-c07a1efe64f6.png)


only port 80 open. let’s see what we can find on the website.

![Screenshot from 2023-01-17 01-20-01](https://user-images.githubusercontent.com/108471951/213867622-9bbd872b-781f-4acf-a521-445e13580711.png)

Nothing found here so i went to README.txt page in the website

192.168.0.106/README.txt

![Screenshot from 2023-01-21 18-24-02](https://user-images.githubusercontent.com/108471951/213867810-e75de70c-d523-40e2-b990-a1bf55c0f43c.png)

We can see its a Joomla Site so I used joomscan

joomscan -u http://192.168.0.106/
  
![Screenshot from 2023-01-21 18-29-02](https://user-images.githubusercontent.com/108471951/213867919-f0ab7340-4334-423a-9a9c-dadabe76f01a.png)

![Screenshot from 2023-01-20 16-02-10](https://user-images.githubusercontent.com/108471951/213867966-7967db14-7ab7-4e02-8dfb-3b5c24414afd.png)

I Nothing found here so I searched for exploit for joomla 3.7.0 exploit in searchsploit

searchsploit joomla 3.7.0


![Screenshot from 2023-01-20 17-37-21](https://user-images.githubusercontent.com/108471951/213868030-e86aae21-c9a0-4e7e-b212-b71f0d2fb760.png)

I found a exploit with exact verison 3.7.0 then i copied the exploit to my path using command 

searchsploit -m php/webapps/42033.txt


![Screenshot from 2023-01-20 17-48-31](https://user-images.githubusercontent.com/108471951/213868147-7a2aa1d5-ab05-4eef-9b96-ccdcea692a76.png)

cat 42033.txt

![Screenshot from 2023-01-20 17-50-09](https://user-images.githubusercontent.com/108471951/213868167-ad422f11-2164-449d-82e3-cdc99bf5b56b.png)

here we can check the usage 

