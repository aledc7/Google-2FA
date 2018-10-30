# Google-2FA
How to implement Two Factor Authenticator in your Linux Server

- [X] Ale DC
- [X] CentOS

## This document explains how to implement 2FA on a Centos server, in my case is in my Docker Server.

### First of all, the google-authenticator package must be installed.
```
yum install google-authenticator
```



### Once installed, I run the program.
```
google-authenticator
```


### the package asks if the token to generate is based on time, it must answer yes.


### then, we will get in response the secret code OTP with its corresponding QR code. (for root user)
```
https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/root@mysecretdomain.com.%3Fsecret%3DGZN7YASDFWERTGMWRFWBBSI%26issuer%mydomain.com
```

![QR 2FA](https://github.com/aledc7/Google-2FA/blob/master/2fa-demo.png "QR for root user")


```
Your new secret key is: ASDFQWERZR3DOMGMUBCPDBSI
Your verification code is 542786
Your emergency scratch codes are:
  45765738
  56515749
  67898406
  75534553
  75333501

```

### Then ask if you want to generate the .google_autenticator file, you must answer yes.
```
Do you want me to update your "/root/.google_authenticator" file? (y/n) y
```
### Then ask if you want to disable multiple uses of the same token, for more security, disable it.
```
Do you want to disallow multiple uses of the same authentication
token? This restricts you to one login about every 30s, but it increases
your chances to notice or even prevent man-in-the-middle attacks (y/n) y
```

### This option is to increase the validity time of the code, it is recommended to answer NO for greater security.
```
By default, a new token is generated every 30 seconds by the mobile app.
In order to compensate for possible time-skew between the client and the server,
we allow an extra token before and after the current time. This allows for a
time skew of up to 30 seconds between authentication server and client. If you
experience problems with poor time synchronization, you can increase the window
from its default size of 3 permitted codes (one previous code, the current
code, the next code) to 17 permitted codes (the 8 previous codes, the current
code, and the 8 next codes). This will permit for a time skew of up to 4 minutes
between client and server.
Do you want to do so? (y/n) n
```

### Limit the time between login attempts
```
If the computer that you are logging into isn't hardened against brute-force
login attempts, you can enable rate-limiting for the authentication module.
By default, this limits attackers to no more than 3 login attempts every 30s.
Do you want to enable rate-limiting? (y/n) y
```
### Here the generation and configuration of PAM ends.
### If all went well, the .google_authenticator file will have been generated


# Now we must configure SSH.

### When configuring ssh always work with two simultaneous open terminals, in case something fails.

### Once we have at least two ssh terminals connected to the server, we edit the sshd files
```
 nano /etc/pam.d/sshd
```

### We must allow users who do not have a token to continue accessing only with password ssh.
### this is achieved by adding this line to the configuration file. "nullok".

```
auth required pam_google_authenticator.so nullok
```

### It is also necessary to comment this line:
```
# auth       substack     password-auth
```


### Then we have to enable 2FA in the SSHD config file.
```
nano /etc/ssh/sshd_config
```

### enable Response Authenticator
```
ChallengeResponseAuthentication yes
#ChallengeResponseAuthentication no
```


### It is also necessary to add this line.
```
AuthenticationMethods publickey,password publickey,keyboard-interactive
```



### Then a code for the user unxs2 is created:

__NOTE__  
if we don't want to generate another code, we could copy the same code of root user.

For this we must copy the .google_Authenticator file in the /home/user folder and then change the owner file whit this command:
```
chown new_owner .google_authenticator   (replace new_owner by the user you want to be the new owner)

```

```
Warning: pasting the following URL into your browser exposes the OTP secret to Google:
https://www.google.com/chart?chs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/mydomain@domain.com%3Fsecret%3DRLBLABLABLAKFURHDSDFMBM%26issuer%3Mydomain.com
```

![QR 2FA](https://github.com/aledc7/Google-2FA/blob/master/2fa-demo.png "QR for unxs2 user")

```
Your new secret key is: LDFGDBFLJRIOJTGADFG;KJFG
Your verification code is 345346
Your emergency scratch codes are:
  45334534
  56756767
  74635633
  56756756
  78457433
```


### Once this is done it is only necessary to restart the services to check the operation (Make sure you have two terminals connected and active).

```
systemctl restart sshd.service
```

### If everything went well, now when you login, you will request the 2Fa code.  


