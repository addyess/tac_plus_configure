# tac_plus_configure
Configuring tac_plus from shrubbery networks

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

## Config file Syntax
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

## Keyword List
### < top_level_decl >
#### key
- help: followed by the tacacs+ encryption key for *all* tacacs+ sessions.  If the end device doesn't know this shared secret, the tacacs+ connection will not occur successfully.  You may only choose one **key** per config file. 
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
#### default authentication file

### < group_decl >
- help: users or other groups may be ganged together into a group. User attributes can be applied as a group to simplify configuration. See the user keyword to see available keywords under group
- syntax: 
```
group = my_group_name {
   # a new group with a sorry name
}
group = my_subgroup_name {
   # inherit all the attributes of my_group_name first
   member = my_group_name
}
```
### < password_spec >
- help: a line of text to specify how the password should be tested
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
 - Run **tac_plus -v** to check for support for ACECLNT, PAM, and SKEY

### < user_decl >
- help: structure used to apply settings to a specific user designated ONLY for tacacs sessions.  
- syntax: 
```
user = my_group_name {
   # a new group with a sorry name
}
```
#### User Attributes
##### name = <string>
##### login = <password_spec>
##### 
#### Service Attributes

