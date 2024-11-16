---
layout: post
---

I have an Android powered TV which support Samba but it doesn't support user/password authentication which means there is no place to input username and password.

>The most common settings are security = share and security = user. If the clients use usernames that are the same as their usernames on the FreeBSD machine, user level security should be used. This is the default security policy and it requires clients to first log on before they can access shared resources.
>
>In share level security, clients do not need to log onto the server with a valid username and password before attempting to connect to a shared resource. This was the default security model for older versions of Samba.

Above words comes from the FreeBSD Handbook, it just indicates *secureity=share * is deprecated, I tried and it doesn't work. So I go back to the [Samba Official Manual](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html) and found the following paremeter:

```
map to guest (G)

This parameter can take four different values, which tell smbd(8) what to do with user login requests that don't match a valid UNIX user in some way.

The four settings are :

Never - Means user login requests with an invalid password are rejected. This is the default.

Bad User - Means user logins with an invalid password are rejected, unless the username does not exist, in which case it is treated as a guest login and mapped into the guest account.

Bad Password - Means user logins with an invalid password are treated as a guest login and mapped into the guest account. Note that this can cause problems as it means that any user incorrectly typing their password will be silently logged on as "guest" - and will not know the reason they cannot access files they think they should - there will have been no message given to them that they got their password wrong. Helpdesk services will hate you if you set the map to guest parameter this way :-).

Bad Uid - Is only applicable when Samba is configured in some type of domain mode security (security = {domain|ads}) and means that user logins which are successfully authenticated but which have no valid Unix user account (and smbd is unable to create one) should be mapped to the defined guest account. This was the default behavior of Samba 2.x releases. Note that if a member server is running winbindd, this option should never be required because the nss_winbind library will export the Windows domain users and groups to the underlying OS via the Name Service Switch interface.

Note that this parameter is needed to set up "Guest" share services. This is because in these modes the name of the resource being requested is not sent to the server until after the server has successfully authenticated the client so the server cannot make authentication decisions at the correct time (connection to the share) for "Guest" shares.

Default: map to guest = Never

Example: map to guest = Bad User
```

I set *map to guest = bad user* and restart it by command `service samba_server restart`, hey it works.

Hereby is my full */usr/local/etc/smb4.conf*, this [link](https://www.server-world.info/en/note?os=FreeBSD_14&p=samba&f=1) is quite helpful too.

```
# cat /usr/local/etc/smb4.conf

[global]
    unix charset = UTF-8
    workgroup = WORKGROUP
    realm = yohoho.home
    netbios name = Rob
    interfaces = bge0 192.168.31.0/24 127.0.0.1
    bind interfaces only = yes
    hosts allow = 192.168.31.0/24

[data]
    path = /data
    public = no
    writable = yes
    printable = no
    guest ok = no
    valid users = jazzi

[movie]
    path = /data/movie
    public = yes
    writable = no
    printable = no
    guest ok = yes
    map to guest = bad user
``` 
