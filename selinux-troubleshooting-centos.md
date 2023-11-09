# Troubleshooting SELinux Centos 

## Troubleshooting a service 

```
## Assumption: Golden Rule of Centos/Redhat 

!!! If everything looks nice (permissions), but NOT START 
it MIGHT BE selinux <-- !!! 

## Step 1: Does service start in permissive mode of selinux  

sestatus
setenforce 0 
# example 
systemctl start systemd-journal-upload 
systemctl status systemd-journal-upload 
# -> Works so, now we know, SELINUX is the problem. 

## Step 2: Findout what go into the way, with smart tools

dnf whatprovides sealert 
dnf install -y setroubleshoot-server 
cd /var/log/audit

# this take a little while - grab some coffee 
sealert -a audit.log > report.txt
```
  * [Alternative way using sealert](selinux-sealert.md) 

```
# now look into the report.txt.
# in most there are 2-3 solutions for you problem 

## Step 3 - possibility 1: Adjust ports/files with semanage command 
# Example 
semanage port -a -t http_port -p tcp 82
semanage port -l | grep http_port 

## Step 3 - possibility 2: A boolean exists 
getsebool -a | grep <boolean_as_mentioned_in_report_txt>
# set boolean permanently (-P) 
# example -> 1 = on or true 
setsebool -P use_virtualbox 1 

## Step 3 - possibility 3: create a module 
# find entries for specific commands 
ausearch -c 'systemd_journal' --raw 
# now create amodule 
ausearch -c 'systemd_journal' --raw | audit2allow -M systemd_journal_fixer 
# now if you want have a look into the module 
cat systemd_journal_fixer.te 

## Step 3 - possibility 3: Install module
semodule -i systemd_journal_fixer.pp 

```

## What is best: setsebool, semanage, create module

  * Best things first 
    1. setsebool (only, if it only opens a small subset of allow-rules) 
    1. semanage 
    1. create module (last resort) 
  * Verify what rules are triggered when using setsebool (might be a lot like in nis_enabled) 
  * Refer to [Using booleans](selinux-boolean.md)


## General 
```
# Find out which problems you had 
cd /var/log/audit 
sealert -a audit.log > report.log

# Alternative - look into messages and find uid 
vi messages
sealert -l de929621-a863-4f2f-ac74-4453138c8c08

# With both you answers how to proceed 
# in case of a port missing 
# e.g. 
# which port type belongs to 80 
semanage port -l | grep 80 
# add you port to that list 
semanage port -a -t http_port_t -p tcp 85

```
