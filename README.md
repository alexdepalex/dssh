dssh
====

Distributed ssh client written in Perl. Standalon so doesn't depend on any libraries etc.


dssh
    dssh - Distributed execution of commands

VERSION
    $Revision: 2.24 $

SYNOPSIS
    dssh [-h]

    dssh [-w *host1,host2* | -W *host1,host2*] [-f *hostfile1,hostfile2*]
    [-q]

    dssh [-w *host1,host2* | -W *host1,host2*] [-f *hostfile1,hostfile2*]
    [-c | [[-a] [-ne] [-no] [-nl] [-np] [-no]] [-g *server group*]
    [-l *login name*] [-t *time out*] *command*

DESCRIPTION
  What is dssh
    dssh stands for distributed secure shell. It's derived from the dsh
    implementation as included by IBM in their PSSP software. The major
    difference is, that dssh supports executing remote commands using SSH
    rather than the unsecure rsh. Furthermore some dsh features are missing
    while other features have been added. dssh Is YABA compatible.

  Command line options
    -h  Help message

    -w *host1,host2*
        Specify one or multiple hosts, separated by commas, on which the
        command should be executed.

    -W *host1,host2*
        Specify one or multiple hosts, separated by commas, on which the
        command should be executed which will be added to the default list.

    -f *hostfile1,hostfile2*
        Specify the host file. Multiple host files should be separated by a
        comma. Use absolute filenames.

    -q  List the hosts. This won't execute any commands.

    -a  Show an activity bar. Will not be shown when -np or -c is specified.

    -l *login name*
        Remote login name. Default is current user name.

    -t *timeout*
        Specify the per process time out. This defaults to 60 seconds.

    -c  Compress output. Emulates the behaviour of dshbak and summarizes the
        output of hosts which have the same output.

    -g *group1,group2*
        Server group. When specified, only processes servers which belong to
        this group. Multiple groups shoud be separated by a comma.

    -ne Don't display standard error output from remote command. Can't be
        used together with -no.

    -no Don't display standard output from remote command. Can't be used
        together with -ne.

    -nh Don't display the hostname in front of every line.

    -nl Removes empty lines from output.

    -np Don't display separators.

A SHORT INTRODUCTION
    Before using dssh, make sure you have distributed your public keys to
    all the hosts you want to access. Furthermore, you need to use an
    unencrypted key (not recommended) or setup ssh-agent (prefered). The
    last option will give you the benefit of using an unencrypted key, while
    the private key is still encrypted.

    The following command is an example of how to execute the command
    "ls -l" on the host nlassv66 using dssh:

    "dssh -w nlassv66 -- ls -l"

    Using the -w option you can specify one or multiple hosts on which you
    want to execute the command. Multiple hosts should be separated by a
    comma. Everything that's typed after the -- won't be evaluated as an
    argument. If you want to be sure that a command won't get evaluated by
    dssh or the shell, place the command between quotes. Example:

    "dssh -w nlassv66,nlassv67 "ls -l""

    When you enter dssh with a command, but without any options:

    "dssh "ls -l""

    dssh will try to locate the default collective file, which is
    "~/.sshcoll", read the hosts which are in the file and execute the
    command on these hosts.

    When you want to specify your own coll file, use the -f parameter:

    "dssh -f /home/<user>/.sshcoll.ce "ls -l""

    The -w options will override the coll file specified by -f.

    You can supply multiple coll files. Seperate them by a comma:

    "dssh -f /home/<user>/.sshcoll.ce,/home/<user>/.sshcoll.li "ls -l""

    When you specify a coll file and you want to add hosts which are missing
    from that file, use the -W options. This will add the specified hosts to
    the host list:

    "dssh -f /home/<user>/.sshcoll.ce -W myhost1,myhost2 "ls -l""

    This will also work:

    "dsh -w myhost1,myhost2 -W myhost3,myhost4 "ls -l""

    If you want to see the hosts on which a command will be executed, use
    the -q option.

    "dssh -f /home/<user>/.sshcoll.ce -W myhost1,myhost2 -q"

    When executing a command on multiple hosts, it might take long before
    dssh is finished. To see if dssh is still active, use to -a option to
    display an activity symbol. Whenever a command gets executed or is
    finished, the symbol will change.

    "dssh -a "ls -l""

    The default user for dssh commands is the current user name. If you want
    your commands to be executed as a different user, specify one using the
    -l option:

    "dssh -l <myuser> "ls -l""

    When a command is executed, a maximum time for executing will be set.
    When this has expired, the command will be killed. The default time is
    60, which might be too short for some commands to finish. Therefore you
    can specify the timout using the -t option:

    "dssh -t 65 "sleep 60""

    On AIX systems which have PSSP installed, there's a program called
    dshbak. This program collects output from the dsh command and
    'compresses' it. When the output for certain hosts is the same, that
    output will only be printed once.

    "dssh -c "ls /usr/bin""

    If you don't want to see the standard error output from a command, use
    the -ne options:

    "dssh -ne "ls -l idontexist""

    If you don't want to see the standard output from a command, use the -no
    options:

    "dssh -no "ls -l""

    Empty lines can be filtered by using the -nl option:

    "dssh -nl "echo '';echo hi;echo ''""

    If you only need the output without the fancy separators, use the -np
    option.

    "dssh -np "ls -l""

    To leave out the hostname, use -nh:

    "dssh -nh "ls -l""

  Groups
    When you have a coll file for a certain customer, it might also be nice
    if there was another way of selecting hosts within that file without
    specifing them on the command line. This can be done using groups. A
    group is an identifier in a coll file, which groups together one or more
    hosts. Lets say that the following output is from our coll file and
    contains 6 hosts.

      host1
      host2
      host3
      host4
      host5
      host6

    Now imagine, that host1 and host2 belong to SAP instance HB1, while
    host3 and host4 are CWS's. When you want to execute a command on the
    CWS's you have three options. Specify the hosts on the command line
    using the -w option, create a separate coll file or create groups in the
    current coll file. We'll take the last option.

      @CWS:host3,host4
      @HB1:host1,host2
      host5
      host6

    If you want to execute a command on all the servers in your coll file,
    you can use dssh without any additional parameters:

    "dssh "ls -l""

    dssh will totally ignore the groups and create a list from all the hosts
    in your coll file. But what if you only want to execute a command on the
    CWS's (host3 and host4)? Then you should use the -g option to specify
    the group:

    "dssh -g CWS "ls -l""

    You can also specify multiple groups. Seperate them by a comma.

    "dssh -g CWS,HB1 "ls -l""

    Furthermore, it's also possible for a host to belong to multiple groups.
    Whenever a command gets executed and a host has multiple entries in the
    coll file, dssh will remove the duplicates internally.

    Another example:

      @CE:host1,host2,host3,host4
      @LI:host5,host6,host7,host8
      @CLASS:host9,host10
      @CECWS:host1,host3
      @LICWS:host5,host8
      @TSM:host8
      @HB1:host1
      @HB2:host2

  User configuration
    A user can create its own personal configuration file in which he can
    set some default settings which affect the behaviour of dssh. These
    settings shoud be placed in <userhome>/.dssh . The format of this file
    is:

    <identifier>=<value>

    Just like other configuration files, the lines can be commented out, or
    a comment can be put in at the end of a line. The user can specify the
    following identifiers:

    Exec *[0|1]*
        Don't execute commands (0)

    timeout *seconds*
        Per process time out in seconds

    hostFile *hostfile*
        Absolute path to the default collective host file

    groups *group1,group2*
        Default host group(s)

    showErr *[0|1]*
        Don't display standard error output (0)

    showOut *[0|1]*
        Don't display standard output (0)

    showHost *[0|1]*
        Don't display hostname at beginning of line (0)

    pretty *[0|1]*
        Don't display separators (0)

    user *username*
        User as which the commands will be execute on the remote hosts

    maxConcurrentChilds *number*
        Set the maximum number of processes that can be running at the same
        time. Don't touch this setting unless you know what your doing !!!

    compress *[0|1]*
        Compress output (1)

    Example

      user=sysmgr
      hostFile=/home/<user>/.sshcoll.int
      compress=1
      groups=CWS,DNS

    When a user issues a dssh command without any additional parameters the
    command will be executed as user sysmgr, on hosts in the file
    /home/<user>/.sshcoll.int which belong to the groups CWS or DNS and
    compressed output.

    bindAddress *IP adress or hostname*
        Host of which the connection will be originating. This is usefull
        for hosts which have multiple network adapter.

AUTHOR
    alexdepalex

KNOWN BUGS
    yes

REPORTING BUGS
    Report bugs to me

COPYRIGHT
    yes

POD ERRORS
    Hey! The above document had some coding errors, which are explained
    below:

    Around line 1660:
        '=item' outside of any '=over'

    Around line 1664:
        You forgot a '=back' before '=head1'

