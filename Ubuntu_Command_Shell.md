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
