# tac_plus_configure
Configuring tac_plus from shrubbery networks

## Table of Contents
- [Reasoning](#reasoning)
- [Current Online Documentation](#current-online-documentation)
- [Best Documentation](#best-documentation)
- [Config File Syntax](#config-file-syntax)
    - [Comments](#comments)
    - [Flag Setting](#flag-setting)
    - [Variable Statement](#variable-statement)
    - [Structure](#structure)
- [Various Lookup Orders](#various-lookup-orders)
    - [Configured Password Order](#configured-password-order)
    - [Password Comparison Order](#password-comparison-order)
    - [Recursive Attribute Order](#recursive-attribute-order)
    - [Key Lookup Order](#key-lookup-order)
    - [Prompt Lookup Order](#prompt-lookup-order)
    - [Enable Password Order](#enable-password-order)
    - [Authorization Algorithm](#authorization-algorithm)
- [Keyword List](#keyword-list)
    - [Top Level Declarations](#-top_level_decl-)
    - [Password Specifications](#-password_spec-)
    - [Host Declarations](#-host_decl-)
    - [Group Declarations](#-group_decl-)
    - [User Declarations](#-user_decl-)
        - [User Attributes](#-user_attr-)
        - [Service Attributes](#-svc-)

## Reasoning
I have found that although there are many references to the mythical documentation of this application... no documentation does a fair enough job detailing how to configure tac_plus from shrubbery networks. 

## Current Online Documentation
- [Ubuntu 15.10 - tacacs+_4.0.4.27a-1](http://manpages.ubuntu.com/manpages/wily/man5/tac_plus.conf.5.html)
- [Ubuntu 14.04LTS - tacacs+_4.0.4.26-3](http://manpages.ubuntu.com/manpages/trusty/man5/tac_plus.conf.5.html)
- [Ubuntu 12.04LTS - tacacs+_4.0.4.19-11build1](http://manpages.ubuntu.com/manpages/precise/man5/tac_plus.conf.5.html)
- [FAQ from Shrubbery.Net](http://www.shrubbery.net/tac_plus/FAQ)
- [Gentoo Wiki](https://wiki.gentoo.org/wiki/Tac_plus#Configuration)

## Best Documentation
The best documentation is unfortunately to look at the code.  Anyone with basic knowledge of the C language shouldn't have much trouble digging through the code to lift out the configuration they want.  So, that's what I'm going to do. If you're reading this and you're not me, you're in luck.  You won't have to read the code. Let's build **tac_plus.conf**

## Config File Syntax
### Comments
Any line that contains a **#** (bang) is a comment beginning at the mark to the end of the line.

The bang can be escaped with a **"** (double quote)
```
# Look, this line in the config file is skipped by the config parser
<keyword(s)> = something_else               # This is a comment now too!!
<keyword(s)> = "#this is not a comment"     # Because of the quotes...
```
### Flag Setting
A statement might be just a keyword (or keyword combination) along on a single line
```
<keyword(s)>
```
### Variable Statement
The value might need to be a single value (eg. string, number, regular expression) or it could be a **structure**.
```
<keyword(s)> = <value>
<keyword(s)> = <value_identifier> {
    ...
}
```
### Structure:
The general structure of the config files are like so:
```
<keyword(s)> = <value_identifier> {
    <statement>
    <statement>
}
```
## Various Lookup Orders
### Configured Password Order
For a given user (and its parent group associations), the password verification method is in this order

1. nopasswrd required
2. the user's pap or login    (depending on the NAS element's request type)
3. the user's global password
4. default authentication file (if available -- even if no user is specified)

If all of these methods fail to find a configured password, then the user is denied
### Password Comparison Order
Due to the way the parsing is handled, your user should settle in on one configured password method.

1. PAM
2. cleartext
3. des
4. file

### Recursive Attribute Order
Attributes for a user or group are checked by:

1. the user being the most specific 
2. it's member group next
3. it's member group next
4. ...

If an attribute is found at one level, it will not recurse down to the next level. 

### Key Lookup Order
The tacacs+ key used to encrypt the session are considered by:
1. the host ip address for host specific key
2. the host name for host specific key
3. the default key

### Prompt Lookup Order
The prompt used to before requesting the username:
1. the host ip address for host specific prompt
2. the host name for host specific prompt

### Enable Password Order
For the 'enable' command, there is an privilege escalation that occurs where a user might need an enable password.

1. SKEY if compiled in and user has attribute **skey**.
2. ACECLNT if compiled in and user has attribute **aceclnt**.
3. Received password is empty string
  1. if not first time requested -- early fail
  2. user has attribute `enable = nopassword` if UENABLE is compiled in
  3. host by hostname has attribute `enable = nopassword` if UENABLE is compiled in
  4. Request for "Password:" is sent
4. Check user or DEFAULTUSER has `enableacl = <matching acl>` is not permitted -- early fail
5. user has attribute `enable = <password_spec>` if UENABLE is compiled in and password matches
6. host by ip has attribute `enable = <password_spec>` or host by hostname and password matches
7. using the login-level **N** the NAS is trying to raise itself to (0-14), look for username $enab**N**$
  1. Uses Configured Password Order for this user if it exists
8. using the username $enable$ or $enab15$ 
  1. Uses Configured Password Order for this user if it exists

### Authorization Algorithm
The complete algorithm by which the daemon processes its configured
AV pairs against the list the NAS sends, is given below.

Find the user (or group) entry for this service (and protocol), then
for each AV pair sent from the NAS:

If the AV pair from the NAS is mandatory:

1.  look for an exact attribute,value match in the user's mandatory list. If found, add the AV pair to the output.
2.  If an exact match doesn't exist, look in the user's optional list for the first attribute match. If found, add the
NAS AV pair to the output.
3.  If no attribute match exists, deny the command if the default is to deny, or,
4.  If the default is permit, add the NAS AV pair to the output.

If the AV pair from the NAS is optional:

5.  look for an exact attribute,value match in the user's mandatory list. If found, add DAEMON's AV pair to output.
6.  If not found, look for the first attribute match in the user's mandatory list. If found, add DAEMONS's AV pair to output.
7.  If no mandatory match exists, look for an exact attribute,value pair match among the daemon's optional AV pairs. If found add the DAEMON's matching AV pair to the output.
8.  If no exact match exists, locate the first attribute match among the daemon's optional AV pairs. If found add the
DAEMON's matching AV pair to the output.
9.  If no match is found, delete the AV pair if the default is deny, or 
10. If the default is permit add the NAS AV pair to the output.

After all AV pairs have been processed, for each mandatory DAEMON AV pair, if there is no attribute match already in the output list, add the AV pair (but add only ONE AV pair for each mandatory attribute).


## Keyword List
### < top_level_decl >
#### key
- help: followed by the tacacs+ encryption key for *all* tacacs+ sessions.  If the end device doesn't know this shared secret, the tacacs+ connection will not occur successfully.  You may only choose one default **key** per config file. 
- syntax: 
```
key = My_Secret_Tacacskey     # this secret has no spaces
key = "My Secret Has Spaces"  # this secret has spaces
```
#### accounting 
- help: path to tacacs+ accounting log file and/or syslog
- syntax: 
```
accounting file = /path/to/accounting/file # messages will be logged with varying formats based on build
accounting syslog                          # messages will be sent to syslog daemon
```
#### logging
- help: syslog facility level
- options: auth, cron, daemon, ftp, kern, local0, local1, local2, local3, local4, local5, local6, local7, lpr, mail, news, syslog, user, uucp
- defaults: **daemon**
- syntax:
```
logging = local6
```
#### default authentication file
- help: read from a valid passwd style database (if /etc/passwd -- will look up in /etc/shadow).  See **Password Verification Order** for more details.  
- defaults: none
- syntax:
```
default authentication = file /path/to/file
```
### < password_spec >
- help: a line of text to specify how the password should be tested
- Run **tac_plus -v** to check for support for ACECLNT, PAM, and SKEY
- syntax:
```
   nopassword             # no password required for this login.
   aceclnt                # password is checked against rsa secure-id library (only available if re-compiled to support it)
   cleartext <password>   # whatever text follows is the cleartext password the user must match
   des <password>         # whatever text follows must match the des version of the user's password
   file <filename>        # read from a valid passwd style database (if /etc/passwd -- will look up in /etc/shadow)
   PAM                    # password is checked against pam library (should be available unless re-compiled out)
   skey                   # one-time password checked against s/key library ( only available if re-compiled to support it)
```
### < host_decl >
- help: attributes applied specifically to certain network elements whose ip address or hostname matches
- warning: through F4.0.4.28 their appears to be a bug using hostnames rather than ip addresses. I submitted a bug report.
- syntax:
```
host = 192.168.0.1 {
   key = My Special Host Key
   prompt = My Special Host Prompt
   enable = cleartext | des | nopassword
}
host = device_supplied_name {
   key = My Special Hostname Key         # THIS APPEARS BROKEN, the key isn't used
   prompt = My Hostname prompt           # Instead the Prompt is used as the key and the prompt
   enable = cleartext | des | nopassword # No password isn't well supported (cfg allows -- doesn't actually work)
}
```
### < group_decl >
- help: users or other groups may be ganged together into a group. User attributes are applied as a group to simplify configuration. See the < user_decl > to see available keywords under group
- syntax: 
```
group = my_group_name {
   # a new group with a sorry name
   <user_attr>*
   <svc>*
}
group = my_subgroup_name {
   # inherit all the attributes of my_group_name first
   member = my_group_name
   <user_attr>*
   <svc>*
}
```
### < user_decl >
- help: structure used to apply settings to a specific user designated ONLY for tacacs sessions.  
- syntax: 
```
user = my_user_name {
   # a new user with a sorry name
   <user_attr>*
   <svc>*
}
```
#### < user_attr >
- help: Use as many of these user attributes inside a user or group structure

##### default service = [ permit | deny ]
- help: Specifies whether the default action for a user is authorized to execute every command or excluded from every command.  You may override with the `cmd = <string> { }` attribute. 
- limits: You may only specify one default service within a block.
- defaults: If not specified, **deny** will be implied
- syntax:
```
 default service = permit
 default service = deny
```
##### name = < string >
- help: Full Name of the User
- syntax:
```
 name = "Johnny Appleseed"
```
##### login = < password_spec >
- help : specifies password used for *authentication*
- syntax:
```
 login = <password_spec>
```
##### member = < group_name >
- help : if any attributes are not defined in this parameter block use the user attributes defined in group = < group_name >
- syntax :
```
 member = my_subgroup_name
```
##### expires = < string >
An entry of the form:
```
  user = lol {
      expires = "MMM DD YYYY"
      password = cleartext "bite me"
  }
```
will cause the user's passwords to become invalid, starting on the expiry date. The only valid date format is e.g. "Jan 1 1980". Case is NOT significant.

A expiry warning message is sent to the user when she logs in, starting at 14 days before the expiration date.

On expiry, the administrator must re-set the expiry date in the configuration file in order to grant continued access. Expiry applies to all password types except "file" passwords.

If passwd(5) files are being used for authentication, the "expires" field in the configuration file is not consulted. Instead, the daemon looks at the "shell" field of the password file entry for a valid expiry date.

If Solaris shadow password files are used for authentication, the "expires" field in the configuration file is not consulted. The expiry field from the shadow password file (if it exists) is used as the expiration date.
##### arap = cleartext < string >
- help : Apple Remote Access Password
- syntax :
```
 arap = cleartext "Apple Password? Really?"
```
##### chap = cleartext < string >
- help : Challenge Handshake Authentication Protocol 
- syntax : 
```
 chap = cleartext "What's up chap?"
```
##### ms-chap = cleartext < string >
- help : Microsoft Challenge Handshake Authentication Protocol 
- syntax : 
```
 ms-chap = cleartext "Because...ya know, Microsoft"
```
##### pap = cleartext < string > | des < string > | PAM
- help : Password Authentication Protocol
- syntax : pick-one
```
 pap = cleartext "Pap in the Clear"
 pap = des XQj4892fjk
 pap = PAM
```
##### opap = cleartext < string > 
- help : Outbound Pap Password
- syntax : 
```
opap = cleartext "outbound pap password"
```
##### global = cleartext < string >
- help : Password for use in all login type : arap, chap, pap, opap, and login
- syntax : 
```
global = cleartext "global password"
```
##### before authorization = < string >
- help : Prior to Authorization, this program will be run as a sub-process.
- Details: The user_guide.in file diagrams the specific details.  Please consult the .tar.gz file from [shrubbery.net](ftp://ftp.shrubbery.net/pub/tac_plus/tacacs-F4.0.4.28.tar.gz) for the gory details.  
- syntax : 
```
 before authorization "/usr/local/bin/pre_authorize $user $port $address"
```
##### after authorization = < string >
- help : After Authorization, this program will be run as a sub-process.
- Details: The user_guide.in file diagrams the specific details.  Please consult the .tar.gz file from [shrubbery.net](ftp://ftp.shrubbery.net/pub/tac_plus/tacacs-F4.0.4.28.tar.gz) for the gory details.  
- syntax : 
```
 # call /usr/local/bin/post_authorize passing it the username, port
 # and current authorization status.
 after authorization "/usr/local/bin/post_authorize $user $port $status"
```

#### < svc >
- help: Use as many of these service attributes inside a user or group structure

##### < svc_auth >
- help: the user authorization request will be sent with AV pairs.  You can permit or deny if the AV pairs sent match these configured.  
- Algorithm : [Authorization Algorithm](#authorization-algorithm)
- default attribute: this directive must be specified first in the block
- keyword(s) : exec, arap, slip, ppp protocol = < proto >, < client defined >
- syntax: 
```
service = < keyword(s) > {
    default attribute = permit
    <attr_value_pair>*
}
```
- attribute value pair
```
    [ optional ] < string > = < string >
    [ optional ] < string > = < string >
    [ optional ] < string > = < string >
```

##### < cmd_auth >
- help: defines the user's authorization to enter a certain command with specified arguments.
- defaults: by default, a command block will deny user authorization
- contents: may only specify `permit <string>` or `deny <string>` where the string argument is an [egrep-style](http://www.cs.columbia.edu/~tal/3261/fall07/handout/egrep_mini-tutorial.htm) regular expression.  
- syntax: 
```
cmd = telnet {
} 
# deny user from using the telnet command

cmd = telnet {
  permit .*
} 
# permits the user to use telnet with any arguments

cmd = telnet {
  permit 192\.168\.0\.[0-9]+
} 
# permits the user to telnet to any host in the 192.168.0.[0-255] network
# denied to telnet to any other argument

cmd = telnet {
  deny 192\.168\.0\.[0-9]+
  permit .*
} 
# denies the user to telnet to any host in the 192.168.0.[0-255] network
# permits to telnet to any other argument


```
