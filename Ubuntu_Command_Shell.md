To list the sudoers listed in the sudo group, I think that the best way to do it would be to run this command (which should be computationally lighter than any of the other commands in this answer):

grep -Po '^sudo.+:\K.*$' /etc/group

Also as suggested in the comments by muru, the format of the entries in /etc/group can be easily handled by cut:

grep '^sudo:.*$' /etc/group | cut -d: -f4

Also again as suggested in the comments by muru, one can use getent in place of grep:

getent group sudo | cut -d: -f4

Any of these commands will print all the users listed in the sudo group in /etc/group (if any).

Command #1 breakdown:

    grep: Prints all the lines matching a regex in a file
    -P: makes grep match Perl-style regexes
    o: makes grep print only the matched string
    '^sudo.+:\K.*$': makes grep match the regex between the quotes

Regex #1 breakdown:

    Any character or group of characters not listed matches the character or the group of characters itself
    ^: start of line
    .+: one or more characters
    \K: discard the previous match
    .*: zero or more characters
    $: end of line

Command #2 breakdown:

    grep: Prints all the lines matching a regex in a file
    '^sudo.+:\K.*$': makes grep match the regex between the quotes
    cut: Prints only a specified section of each line in a file
    -d:: makes cut interpret : as a field delimiter
    -f4: makes cut print only the fourth field

Regex #2 breakdown:

    Any character or group of characters not listed matches the character or the group of characters itself
    ^: start of line
    .*: zero or more characters
    $: end of line




Expanding on the sudo -l -U test, one can use getent passwd to determine the users who can use sudo. Using getent allows us to access users who may not be present in the passwd file, such as LDAP users:

getent passwd | cut -f1 -d: | sudo xargs -L1 sudo -l -U | grep -v 'not allowed'

sudo -U does not return a non-zero exit value that we could take advantage of, so we are reduced to grepping the output.

This command returns a list of users with sudo rights:

awk -F ":" '{ system("groups " $1 " | grep -P \"[[:space:]]sudo([[:space:]]|$)\"") }' /etc/passwd
Output is (e.g.):

<username> : <username> adm cdrom sudo dip plugdev lpadmin sambashare docker

If only the user name to be displayed, then this command:

awk -F ":" '{ system("groups " $1 " | grep -P \"[[:space:]]sudo([[:space:]]|$)\"") }' | awk -F ":" '{ print $1 }' /etc/passwd

Use command

#visudo 

will allow an administrator to inspect and amend the privileges of groups that can use the sudo command. 

To list all local users you can use:

cut -d: -f1 /etc/passwd

To list all users capable of authenticating (in some way), including non-local, see this reply: http://askubuntu.com/a/414561/571941

Some more useful user-management commands (also limited to local users):

To add a new user you can use:

sudo adduser new_username

or:

sudo useradd new_username

See also: What is the difference between adduser and useradd?

To remove/delete a user, first you can use:

sudo userdel username

Then you may want to delete the home directory for the deleted user account :

sudo rm -r /home/username

(Please use with caution the above command!)

To modify the username of a user:

usermod -l new_username old_username

To change the password for a user:

sudo passwd username

To change the shell for a user:

sudo chsh username

To change the details for a user (for example real name):

sudo chfn username
