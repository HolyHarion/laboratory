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

 30
down vote
	

The easiest way to get this kind of information is getent - see manpage for the getent command Manpage icon. While that command gives the same output as cat /etc/passwd it is useful to remember because it will give you lists of several elements in the OS.

To get a list of all users you type (as users are listed in /etc/passwd)

getent passwd

To add a user newuser to the system you would type

sudo adduser newuser

to create a user that has all default settings applied.

Bonus: To add any user (for instance anyuser) to a group (for instance cdrom) type

sudo adduser anyuser cdrom

You delete a user (for instance obsolete) with

sudo deluser obsolete

If you want to delete his home directory/mails as well you type

sudo deluser --remove-home obsolete

And

sudo deluser --remove-all-files obsolete

will remove the user and all files owned by this user on the whole system.



list of all users who can login (no system users like: bin,deamon,mail,sys, etc.)

awk -F':' '$2 ~ "\$" {print $1}' /etc/shadow

add new user

sudo adduser new_username

or

sudo useradd new_username

delete/remove username

sudo userdel username

If you want to delete the home directory (default the directory /home/username)

sudo deluser --remove-home username

or

sudo rm -r /path/to/user_home_dir

If you want to delete all files from the system from this user (not only is the home diretory)

sudo deluser --remove-all-files



This should get, under most normal situations, all normal (non-system, not weird, etc) users:

awk -F'[/:]' '{if ($3 >= 1000 && $3 != 65534) print $1}' /etc/passwd

This works by:

    reading in from /etc/passwd
    using : as a delimiter
    if the third field (the User ID number) is larger than 1000 and not 65534, the first field (the username of the user) is printed.

This is because on many linux systems, usernames above 1000 are reserved for unprivileged (you could say normal) users. Some info on this here:

    A user ID (UID) is a unique positive integer assigned by a Unix-like operating system to each user. Each user is identified to the system by its UID, and user names are generally used only as an interface for humans.

    UIDs are stored, along with their corresponding user names and other user-specific information, in the /etc/passwd file...

    The third field contains the UID, and the fourth field contains the group ID (GID), which by default is equal to the UID for all ordinary users.

    In the Linux kernels 2.4 and above, UIDs are unsigned 32-bit integers that can represent values from zero to 4,294,967,296. However, it is advisable to use values only up to 65,534 in order to maintain compatibility with systems using older kernels or filesystems that can only accommodate 16-bit UIDs.

    The UID of 0 has a special role: it is always the root account (i.e., the omnipotent administrative user). Although the user name can be changed on this account and additional accounts can be created with the same UID, neither action is wise from a security point of view.

    The UID 65534 is commonly reserved for nobody, a user with no system privileges, as opposed to an ordinary (i.e., non-privileged) user. This UID is often used for individuals accessing the system remotely via FTP (file transfer protocol) or HTTP (hypertext transfer protocol).

    UIDs 1 through 99 are traditionally reserved for special system users (sometimes called pseudo-users), such as wheel, daemon, lp, operator, news, mail, etc. These users are administrators who do not need total root powers, but who perform some administrative tasks and thus need more privileges than those given to ordinary users.

    Some Linux distributions (i.e., versions) begin UIDs for non-privileged users at 100. Others, such as Red Hat, begin them at 500, and still others, such Debian, start them at 1000. Because of the differences among distributions, manual intervention can be necessary if multiple distributions are used in a network in an organization.

    Also, it can be convenient to reserve a block of UIDs for local users, such as 1000 through 9999, and another block for remote users (i.e., users elsewhere on the network), such as 10000 to 65534. The important thing is to decide on a scheme and adhere to it.

    Among the advantages of this practice of reserving blocks of numbers for particular types of users is that it makes it more convenient to search through system logs for suspicious user activity.

    Contrary to popular belief, it is not necessary that each entry in the UID field be unique. However, non-unique UIDs can cause security problems, and thus UIDs should be kept unique across the entire organization. Likewise, recycling of UIDs from former users should be avoided for as long as possible.


