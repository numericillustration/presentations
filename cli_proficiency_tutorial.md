# Mostly Bash tricks

Some CLI tips, tricks and bash things to save you typing and
hopefully not all in terrible ways

Many times you start with some command that turns into a more than 80 chars
one liner that turns into the double loop one liner from hell with no error
checking.  While that has its place, I'd rather help you turn that into a great
script!

## prep

1. Get a Linux VM to play on if desired

    For total safety, you can do anything to a VM and even if you accidentally destroy it, no big deal.  Make another one.

    - Vagrant:
    If you have an Intel Mac this will work.  "Apple Silicon" users this fails
    - go to `https://www.vagrantup.com` and download
    - install Virtualbox or the (VMWare Fusion plugin)
    - run a VM

        ```bash
        vagrant init hashicorp/bionic64
        vagrant up
        vagrant ssh
        ```

        NOTE: Apple silicon users be advised, you need to use arm images and will
        need a release of VirtualBox or VMWare Fusion for arm (Appe silicon is an
        ARM platform)

    - Triton (RIP)

        ```bash
        triton -p iad001 instance create --wait --name=my_sandbox  centos-7  g4-highcpu-4G
        triton -p iad001 instance ssh my_sandbox
        ```

    - Fusion if you must
    - UTM
    - get [the free download][31]
    - follow [the Debian UTM guide][https://docs.getutm.app/guides/debian/]
2. ssh into whatever host you setup

    ```bash
    2024.09.25.13:29:23 ~/gits/presentations 🂁 ssh -o PreferredAuthentications=keyboard-interactive,password -o PasswordAuthentication=yes 192.168.64.4
    The authenticity of host '192.168.64.4 (192.168.64.4)' can't be established.
    ED25519 key fingerprint is SHA256:39fAgENKVOwu/kWBAEixQFDAtGWYva3I94tHHu+44UA.
    This key is not known by any other names.
    Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
    Warning: Permanently added '192.168.64.4' (ED25519) to the list of known hosts.
    mhicks@192.168.64.4's password:
    Linux clitutorial 6.1.0-25-arm64 #1 SMP Debian 6.1.106-3 (2024-08-26) aarch64

    The programs included with the Debian GNU/Linux system are free software;
    the exact distribution terms for each program are described in the
    individual files in /usr/share/doc/*/copyright.

    Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
    permitted by applicable law.
    Last login: Wed Sep 25 16:28:18 2024
    mhicks@clitutorial:~$
    ```
3. Make a play directory

   ```bash
   mkdir cli_tutorial
   cd cli_tutorial
   ```

## BASICS

1. [io redirection][1]

    Files, file descriptors and IO redirection:

    > There are always three default files open, stdin (the keyboard), stdout
    (the screen), and stderr (error messages output to the screen). These, and
    any other open files, can be redirected. Redirection simply means capturing
    output from a file, command, program, script, or even code block within a
    script and sending it as input to another file, command, program, or script.
    Each open file gets assigned a file descriptor. The file descriptors for
    stdin, stdout, and stderr are 0, 1, and 2, respectively. For opening
    additional files, there remain descriptors 3 to 9.

     (from [ch 20 of the Advanced Bash Scripting guide][29].

    Redirection operators:

    - `|` pipe - takes stdout of the left and sends to stdin of the right
                ( like irrigation pipes )
    - `>` send output to beginning of file or io handle ( can call this a funnel)
    - `>>` append output to file or io handle
    - `< filename` accept input from file
    - `&>` redirect stdout and stderr ( newish )
    - `[j]<>filename` open file for reading and writing with file descriptor j

    Common uses:

    1. send stderr of a commant to "the trash"

        useful when you don't want to see standard error

        ```bash
        command 2>/dev/null
        ```

    2. send stdout and stderr to the trash

        useful when you don't want to see any output

        ```bash
        command >/dev/null 2>&1 # old
        command &>/dev/null     # as of bash 4 final and newer
        ```

        note: the order of `>` and `2>&1` are important

    3. send stderr of a command to stdout.

        Useful when you want to be able to get all output as stdout to grep it.

        ```bash
        command 2>&1
        ```

    4. append stdout of a command to a file

        ```bash
        command >>filename
        ```

    5. Accept input from a file

        While I'm sure there are other good uses of this, my most common use is
        to read lines from a file in a loop.  More on loops later

        ```bash
        while read this_line
        do
           echo "line contents found were [$this_line]"
        done < filename
        ```

        example:

        ```bash
        🂁 echo foo>foo.txt
        🂁 echo bar>>foo.txt
        🂁 echo bar1>>foo.txt
        🂁 echo bar2>>foo.txt
        🂁 while read this_line
        do
           echo "line contents found were [$this_line]"
        done < foo.txt
        line contents found were [foo]
        line contents found were [bar]
        line contents found were [bar1]
        line contents found were [bar2]
        ```

    6. Some honorable mention to tee

        with pipe and funnel, tee is also a "plumbing" term
        You take one input and duplicate it out 2 outputs.  The following
        will take the stdout of `command` and send it to a file and stdout.

        ```bash
        command | tee filename
        ```

        `tee` also has the `-a` flag for appending like `>>`

        My most common use for this is where I need elevated permissions to
        write to a file.  Important to know that  in constructs like:

        ```bash
        sudo command >filename
        ```

        the sudo permissions to run `command` do not carry over through the `>`
        to `filename` so you will have to do something like

        ```bash
        sudo command >temp_filename
        cat filename temp_filename > third_file
        sudo mv third_file filename
        ```

        or alternatively  you can use tee like

        ```bash
        sudo command | sudo tee filename 1>/dev/null
        ```

2. style and readability

    ```bash
    echo "Captain: What happen ? Mechanic: Somebody set up us the bomb. (spoken in the Flash animation as Someone set up us the bomb) Operator: We get signal. Captain: What ! Operator: Main screen turn on. Captain: It's you !! CATS: How are you gentlemen !! CATS: All your base are belong to us. CATS: You are on the way to destruction. Captain: What you say !! CATS: You have no chance to survive make your time. CATS: Ha Ha Ha Ha ...." | sort | uniq | wc -l | xargs |.....
    ```

    Bash allows one to escape the newline character in strings.  So this looooooong line can
    be rewritten neatly on multiple lines to make it much more readable and maintainable.

    ```bash
    echo "Captain: What happen ?\
          Mechanic: Somebody set up us the bomb. (spoken in the Flash animation as Someone set up us the bomb)\
          Operator: We get signal.\
          Captain: What !\
          Operator: Main screen turn on.\
          Captain: It's you !!\
          CATS: How are you gentlemen !!\
          CATS: All your base are belong to us.\
          CATS: You are on the way to destruction.\
          Captain: What you say !!\
          CATS: You have no chance to survive make your time.\
          CATS: Ha Ha Ha Ha ...."\
            | sort \
            | uniq \
            | wc -l \
            | xargs \
            |.....
    ```

    see also [Zero Wing][33]

3. Composability and the unix philosophy

    Each thing should do one job well and these components can be combined
    together to do complicated tasks by piping the output of one command to
    the input of another.

    the operator is:

    ```text
    |
    ```

    ```bash
    echo "all your base" | wc -c
    security find-generic-password \
               -s tritonproduct.1password.com \
               -w \
               | op signin --account tritonproduct
    ```

4. [Variables][2] - keeping things to use again - simple assignment

    ```bash
    foo='stuff'
    foo="stuff with $other_variables expanded"
    foo="stuff with variables with .surrounding_${stuff}smashed into it"
    ```

5. HISTORY

    "I know I did this before..."
    - the Bash manual has some [info on settings][24] and [heres what I use][9]
    - searching the history

        ```bash
        CTRL+r <type>
        ```

        ```bash
        history | grep foo
        ```

6. bash [keybinding/shortcuts][4]

    native is `emacs`, the most important ones to me are:

    ```bash
    control+e   # jump to the end of the line
    control+a   # jump to the beginning of the line
    control+w   # erase back to beginning of 1 word
    ```

## intermediary CLI things

1. [testing][3] strings, output, numbers

    OLD: `[ ]`
    NEW: `[[ ]]` and `(( ))`

    ```bash
    if [[ -z $myvariable ]]
    then
        echo "myvariable is empty"
    fi
    ```

    ```bash
    if (( foo >= 5 ))
    then
    ...
    ```

2. [list constructs][6]

    ```bash
    true && echo "yeppers" || echo "nope"
    false && echo "yeppers" || echo "nope"
    ```

    ```bash
    (($(zoneadm list -vc | grep -v global | wc -l))) && echo "Rebooting" && shutdown -y -g0 -i6 || echo "Reboot refused - CN has instances."
    ```

3. WAT is my script doing??  What is this other script doing? aka [tracing][30]

    If you're stuck iand don't want to or can't add 1000 echo/printf lines

    ```bash
    set -o xtrace
    bash -x /foo.sh
    ```

4. loops

    > a loop is a block of code that iterates a list of commands as long
      as the loop control condition is true

    Often I want to [iterate over a list][29] of things.

    `for` loops are simple

    ```bash
    for my_var in foo bar baz
    do
        echo "look ma, I have $my_var"
    done
    ```

    `while` loops are a little more complicated

    ```bash
    while <some condition>
    do
        ...
    done
    ```

5. what even is this printf

    Unfortunately sometimes echo is just okay.

    Printf has many much options for consistent output

    1. outputting content [as][26] [json][25]

        ```bash
        if ((FIRST == 0))
        then
            printf "\n%b" "    { \"uuid\": \"$uuid\", \"state\": \"${zones[$uuid]%%λλλ*}\", \"zone_state\": \"${zones[$uuid]##*λλλ}\" }"
            let "FIRST += 1"
        else
            printf ",\n%b" "    { \"uuid\": \"$uuid\", \"state\": \"${zones[$uuid]%%λλλ*}\", \"zone_state\": \"${zones[$uuid]##*λλλ}\" }"
        fi
        ```

        ```bash
        printf -- "-e 'this[\"%s\"][\"%s\"][\"%s\"][\"%s\"]=%s' -e 'this[\"%s\"][\"%s\"][\"%s\"][\"%s\"]=%s' " \
                        "$server_uuid" \
                        "$component" \
                        "$this_shard" \
                        "$component_image_from" \
                        '0' \
                        "$server_uuid" \
                        "$component" \
                        "$this_shard" \
                        "$component_image_to" \
                        "$((instance_count + already_existing_image_to_instance_count))"
        ```

    2. [tabular output][27]

        ```bash
        printf '%-47s %-36s %-24s %-32s %-12s %s\n' "$this_fingerprint" "$this_uuid" "$this_login" "$this_email" "$this_approved_for_provisioning" "$this_memberof"
        ```

6. avoid useless use of cat

    ```bash
    cat file | grep foo
    cat file | awk '{print $1}'
    ```

    many of the utilities that take stdin for other `|` pipelining also
    take a filename as an argument.

    ```bash
    grep foo file
    awk '{print $1}' file
    ```

## Advanced

1. Running chunks of code: subshells, blocks, and functions

    Often in shell scripts one runs external commands.  But when you *just*
    want to run some shell commands without writing them out to a file, and
    executing that, you can subshell.

    Reasons:

    - reuse and legibility
    - encapsulation
    - capturing output AND exit codes

    1. Functions:

        ```bash
        if (( DEBUG ))
        then
            printf 'DEBUG - blah\n'
        fi
        ```

        You may want to refactor into

        ```bash
        function log () {
            (( DEBUG )) && printf 'DEBUG - %s\n' "$1" >&2
        }

        # later in the code
        log "blah blah blah"
        ```

    2. Subshells:

        ```bash
        🂁 my_rand="$(( RANDOM % 100 ))"; printf 'me is %s parent is %s rand is %s\n' "$BASHPID" "$$" "$my_rand"
        me is 12915 parent is 12915 rand is 11262
        # note ^- pid is same -^

        🂁 ( my_rand="$(( RANDOM % 100 ))"; printf 'me is %s parent is %s rand is %s\n' "$BASHPID" "$$" "$my_rand")
        me is 13020 parent is 12915 rand is 7849
        # note ^- pid different -^
        ```

        ```bash
        🂁 myrand="$(( RANDOM % 100 ))"
        🂁 echo $myrand
        56
        🂁 ( (( myrand++ )) ; echo $myrand )
        57
        🂁 echo $myrand
        56
        🂁 ( for i in {1..4}} ; do (( myrand++ )) ; echo $myrand ; done )
        57
        58
        59
        60
        🂁 echo $myrand
        56
        ```

    3. Capturing output from subshells

        NOTE -The older back-tic style is not as clear that its string output

        ```bash
        `foo`
        ```

        as the newer stype

        ```bash
        $(foo)
        ```

        Where its obvious you are doing bash variable-esque (all variables are strings) expansion of the output.

    4. capturing output and the exit code

        With a subshell, if you want the output and [optionally the return code][7] you
        can get both.

        ```bash
        diff file1 file2
        ```

        will tell you the diff, or output nothing.  Did it succeed?

        ```bash
        diff_output="$(diff file1 file2)"
        ret="$?"
        if (( ret ))
        then
            printf 'Files differ\n%s\n' "$diff_output"
        else
            printf 'files identical\n'
        fi
        ```

    5. saving the extra overhead of a full subshell exec

        You can also use brace notation for nonsubshell encapsulation

        ```bash
        🂁 myrand="$(( RANDOM % 100 ))"
        🂁 echo $myrand
        56
        🂁 { for i in {1..4}} ; do (( myrand++ )) ; echo $myrand ; done }
        57
        58
        59
        60

        ```

        What do you think will output now if we echo myrand ?  See the later section on scope.

2. pasting blocks of commands

    Some commands ask for input from the user.  If you paste in commands after
    them, intead of prompting you, they may take subsequent commands as
    that inout, where what you want is for it to recognize those as commands
    to run afterwards and still prompt you.  Pasting in a curly brace code
    block will cause bash to interpret the multiple commands properly

    ```bash
    {
        foo
        foo2
        foo3
    }
    ```

3. [read][35] for parsing standardized lines

    When you have a known format for some lines you want to parse, often an easy way
    to handle that output is to read into multiple variables all at once

    ```bash
    myline='foo bar baz buz'
    ```

    instead of trying to use regex, awk, or cut to parse it you can use read:

    ```bash
    read var1 var2 var3 var4 <<<"$myline"
    ```

    [example][15]:

    or with different delimiters

    ```bash
    myline2='foo,bar,baz,buz'
    {
    IFS=,
    read var1 var2 var3 var4 <<<"$myline2"
    }
    ```

    [bash IFS docs][36]

4. Advanced IO redirection

    1. subshells as a file handle

        ```bash
        <( foo ) for reading output as a filehandle
        ```

        [diffing some pretty formatted json from 2 unpretty files][11]

        ```bash
        diff -u <(json -f "$cm-pre-$component-$date.json") <(json -f "$cm-post-$component-$date.json")
        ```

        diffing local file vs remote file ( via "output of a command on another host via ssh" as a file )

        ```bash
        diff -u /etc/ssh/sshd_config <(ssh admin.headnode.iad001.joyent.us 'cat /etc/ssh/sshd_config')
        ```

        Read lines of output from a command as a file without saving it anywhere -- from [this zookeeper tooling][12]

        ```bash
        while read line
        do
            # stuff done with $line
        done < <(echo mntr | nc -w5 ${ZK_IP} 2181)
        ```

    2. Redirecting contents of a variable to stdin ( an avoiding useless use of cat technique )

        This is done with the `<<<` operator

        ```bash
        VMINFO=$($VMAPI /vms/$vm_uuid | $JSON -Ha)
        vm_owner_uuid=$($JSON -Ha owner_uuid <<< "$VMINFO")
        state=$($JSON -Ha state <<< "$VMINFO")
        vm_brand=$($JSON -Ha brand <<< "$VMINFO" )
        ```

5. [arrays][5]

    A Single name with multiple indices to reference contents

    This can be handy to handle [multiline output from commands][13]
    ex from [torwatch][13]

    1. creating them

        Use the `mapfile` or `readarray` keyword

        ```bash
        readarray -t my_array <<<"$(/opt/smartdc/bin/sdc-server lookup --hostname rack_identifier=~"$this_rack_identifier")"
        ```

        Also you can hand define arrays line by line with `()`

        ```bash
        declare -a my_array
        my_array=( 'foo' 'bar' 'baz' )
        ```

        example, when you have a bunch of [messy stuff you want/need
        to single quote][10]

        ```bash
        declare -a my_array=(
            '{ "and" : [ { "eq": ["state", "failed" ] }, { "ne": ["zone_state", "destroyed" ] }  ] }'
            '{ "eq": ["state", "incomplete" ] }'
            '{ "eq": ["state", "shutting_down" ] }'
            '{ "eq": ["state", "provisioning" ] }'
            '{ "eq": ["state", "down" ] }'
        )
        ```

    2. access the contents of the array

        Arrays are "indexed", they have content at numbered "slots" starting at
        zero.

        to get the contents in the first (0th) slot

        ```bash
        ${my_array[0]}
        ```

        To get the contents of all the array at once

        ```bash
        ${my_array[@]}
        ${my_array[*]}
        ```

        getting a subset of elements

        ```bash
        start=2
        end=5
        echo "${my_array[@]:$start:$end}"
        ```

    3. get a count of used indices

        ( Technically this is returning the next available index, due to
        starting the index at zero )

        ```bash
        ${#my_array[@]}
        ```

    4. combining arrays

        ```bash
        declare -a new_array
        new_array=( ${array1[@]} ${array2[@]} )
        ```

    5. loop over array indices using the count of used indices

        ```bash
        for (( i=0;  i < ${#my_array[@]}; i++ ))
        do
            echo "I found at index $i ${my_array[$i]}"
        done
        ```

    6. checking arrays for content

        ```bash
        function does_array_x_contain_y () {
            local -n arrayX=$1
            local item=$2
            echo "${arrayX[@]}" | grep -q "$item"
            local retval=$?
            (( retval )) && IN='is NOT' || IN='IS'
            log "$LINENO we checked and item $item $IN in this array"
            return "$retval"
        }
        ```

        from [torwatch][14]

6. [Heredocs][8] - when you have a lot to output and echo just isn't cutting it

    See  ops-cnsetup script for [heredoc with variable expansion][28]

    ```bash
    cat >somefile <<EOF
    echo "${myvariable}"
    EOF
    ```

    ```bash
    cat > somefile <<'EOF'
    echo "${myvariable}"
    EOF
    ```

7. scope of variables

    its all global unless you use local

    ```bash
    this_instance='foo'
    printf 'global this_instance is: %s\n' "$this_instance"
    function fooinate() {
        local this_instance="$1"
        printf 'inside fooinate, local this_instance starts as: %s\n' "$this_instance"
        this_instance="$this_instance blah blah blah"
        printf 'inside fooinate, but now local this_instance modified to: %s\n' "$this_instance"
    }

    fooinate "$this_instance"
    printf 'global this_instance is: %s\n' "$this_instance"
    ```

8. Efficiency of staying in BASH

    While composability with other tools is great, often there are bash builtins
    for many of the things one wants to do and there are some savings on overhead
    (runtime, memory, file descriptor use...) which can be gained by NOT shelling
    out to other utilities.

    1. regex string match right in your script instead of piping to grep

        ```bash
        if echo "$NODE" | grep -E '[R][ABC][A-Z0-9]{4,6}' ;then
           USERNAME="ADMIN"
        elif echo "$NODE" | grep -E '[MS|RM][A-Z0-9]{4,6}' ;then
           USERNAME="ADMIN"
        elif echo "$NODE" | grep -E '^[A-Z0-9]{7}$' ; then
           USERNAME="root"
        fi
        ```

        The bash regex match operator is `=~`

        ```bash
        RICH_REGEX='[R][ABC][A-Z0-9]{4,6}'
        MANTA_REGEX='[MS|RM][A-Z0-9]{4,6}'
        DELL510_REGEX='^[A-Z0-9]{7}$'

        if [[ $NODE =~ $RICH_REGEX ]];then
           USERNAME="ADMIN"
        elif [[ $NODE =~ $MANTA_REGEX ]];then
           USERNAME="ADMIN"
        elif [[ $NODE =~ $DELL510_REGEX ]]; then
           USERNAME="root"
        fi
        ```

        this example is from [here][16]

    2. case changes

        There are a large number of ways you can shell out to do this:

        *tr* example

        ```bash
        username="$(echo "$username" | tr '[:lower:]')"
        ```

        *perl*

        ```bash
        username="$(echo "$username" |  perl -ne 'print lc')"
        ```

        But you can also do it right in bash >= 4.0

        ```bash
        username="${username,,}"
        ```

        or to *upper* case:

        ```bash
        username="${username^^}"
        ```

9. background processing

    You can background long running processes and let them run while you do
    other stuff.

    ```bash
    if [[ $(find "$PLUGIN_TMP_FILE" -mmin +"$REFRESH_INTERVAL") ]]
    then
        cp "$PLUGIN_TMP_FILE" "$PLUGIN_OUT_FILE"
        fetch_manatee_adm_out "$PLUGIN_TMP_FILE" &
    fi

    parse_manatee_adm_out "$PLUGIN_OUT_FILE"
    ```

    from `/opt/custom/telegraf/bin/120s/manatee-adm.sh`

## Habits

1. scratchpad

    keep useful 1 liners in a file

    I cannot tell you how many times I have dug these up and reused them
    with minor alteration or even converted them into [common tooling][21]
    I can [share with the team][22] or that [lands in docs][23].

2. put all the commands used to get useful info *in the JIRA ticket* not just the output.

## profile things

1. aliases

    you can save a lot of keystrokes for things you want to do
    a certain way by aliasing

    ```bash
    alias ssh='ssh -A'
    alias less='less -wM --tabs=3'
    ```

2. profile functions

    a terminal [countdown timer][17] to use in place of sleep
    for when you want to see it happen

    ```bash
    countdown() {
        local seconds="$1"
        local outro="$2"

        if [[ -z $outro ]]
        then
            outro='done'
        fi
        while (( seconds > 0 ))
        do
            printf '\r%s%s ' "$(tput el)" "$((seconds--))"
            sleep 1
        done
        printf '\r%s%s\n' "$(tput el)" "$outro"
    }
    ```

    - [print out log entries only when in debug mode][18]
    - [a collection of these][34]

3. CDPATH

    A way to save some keystrokes on typing the full path to dirs you visit regularly

    ```bash
    # probe all these paths for existence and add to CDPATH
    MY_CDPATHS="
    $HOME/Documents/
    $HOME/gits/joyent/
    "
    for cdpath in $MY_CDPATHS; do
         test -d "$cdpath" && CDPATH="$CDPATH:$cdpath"
    done
    export CDPATH
    ```

    ```bash
    ~ 🂁 cd opstools/
    /Users/michaelhicks/gits/joyent/opstools
    ```

## SSH local client configs

You can create and keep local settings in
`.ssh/config`

favorites:

1. persistent and multiplexed sessions `man ssh_config` and see ControlMaster

    ```text
    ControlMaster auto
    ControlPath ~/.ssh/controlmasters/%C
    #ControlPath ~/.ssh/controlmasters/%h:%p_%r
    #    include at least %h, %p, and %r (or alternatively %C)
    #           %%    A literal `%'.
    #           %C    Hash of %l%h%p%r.
    #           %d    Local user's home directory.
    #           %h    The remote hostname.
    #           %i    The local user ID.
    #           %L    The local hostname.
    #           %l    The local hostname, including the domain name.
    #           %n    The original remote hostname, as given on the command line.
    #           %p    The remote port.
    #           %r    The remote username.
    #           %T    The local tun(4) or tap(4) network interface assigned if tunnel forwarding was requested, or "NONE" otherwise.
    #           %u    The local username.

    ControlPersist 720
    ```

2. set a username for some hosts

   ```text
    Host devops.int.joyent.us
        User mhicks
        Hostname 10.55.0.8
        ForwardAgent yes
        ProxyJump root@64.30.130.94
    ```

## other utilities I wish I knew about years ago

`json` or `jq`

`grep -r`

`grep -c`  (when available rather than grep | wc -l)

`less +G`

`less +F`

## adding extra Key Bindings in your profile

1. make up and down do Ctrl-R like searching based on what is partially typed

    ```bash
    bind '"\e[A": history-search-backward'
    bind '"\e[B": history-search-forward'
    ```

## vim things

1. config file [`.vimrc`][19]

2. visual mode

   I mostly use this to block edit

3. `*` search shortcut

4. [regex][20]

## Optimizations

1. re-calling utilities for info you already have

   Read the output once into a variable and then repeatedly pipe
   through parsers such as json, grep, awk... for the parts you want to capture.

   ```bash
   smart_output="$(smartctl -a /dev/sda)"


    unsup_match="$(grep -E 'SMART support is:.*Unavailable - device lacks SMART capability.|SMART support is:.*Disabled' <<< "$smart_output")"

    disk_serial="$(awk '($1~/Serial/){print $3}' <<< "$smart_output")"

   ```

[1]: https://tldp.org/LDP/abs/html/io-redirection.html
[2]: https://tldp.org/LDP/abs/html/variables.html
[3]: https://tldp.org/LDP/abs/html/testconstructs.html
[4]: https://gist.github.com/tuxfight3r/60051ac67c5f0445efee
[5]: https://www.gnu.org/software/bash/manual/bash.html#index-mapfile
[6]: https://tldp.org/LDP/abs/html/list-cons.html
[7]: https://gist.github.com/numericillustration/7a45b19ee43512a475bd83de881e3333#file-disk_manager-sh-L1391-L1398
[8]: https://tldp.org/LDP/abs/html/here-docs.html
[9]: https://github.com/numericillustration/dotfiles/blob/master/profile#L100-L113
[10]: https://gist.github.com/numericillustration/c2e39c6b9473675cacc78ddb79f90d38#file-pager3-sh-L29-L35
[11]: https://github.com/joyent/change-mgmt/blob/master/tools/manta-component-updater.sh#L501
[12]: https://github.com/joyent/opstools/blob/master/src/bin/manta_monitoring_zookeeper.sh#L55
[13]: https://github.com/joyent/change-mgmt/blob/master/tools/torwatch.sh#L527
[14]: https://github.com/joyent/change-mgmt/blob/master/tools/torwatch.sh#L40-L49
[15]: https://gist.github.com/numericillustration/c2e39c6b9473675cacc78ddb79f90d38#file-pager3-sh-L60
[16]: https://github.com/joyent/opstools/blob/master/src/bin/jpc-ipmi#L61-L88
[17]: https://github.com/numericillustration/dotfiles/blob/master/profile#L366-L381
[18]: https://github.com/joyent/change-mgmt/blob/master/tools/manta-component-updater.sh#L24-L33
[19]: https://github.com/numericillustration/dotfiles/blob/master/vimrc
[20]: http://vimregex.com
[21]: https://github.com/joyent/opstools/blob/master/src/bin/ufds_user_disable.sh
[22]: https://github.com/joyent/NOCTools
[23]: https://github.com/joyent/ops-sop/blob/master/docs/ops/ipmi.md#discovering-current-password
[24]: https://www.gnu.org/software/bash/manual/html_node/Bash-History-Facilities.html
[25]: https://github.com/joyent/change-mgmt/blob/master/tools/manta-component-updater.sh#L453-L464
[26]: https://gist.github.com/numericillustration/c2e39c6b9473675cacc78ddb79f90d38#file-pager3-sh-L99-L108
[27]: https://github.com/joyent/opstools/blob/master/src/bin/triton-user-keytrace#L107
[28]: https://github.com/joyent/opstools/blob/master/src/bin/ops-cnsetup#L1158-L1177
[29]: https://tldp.org/LDP/abs/html/loops1.html
[30]: https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#The-Set-Builtin
[31]: https://mac.getutm.app
[32]: https://ubuntu.com/download/server/arm
[33]: https://en.wikipedia.org/wiki/Zero_Wing
[34]: https://github.com/joyent/devops-tools/pull/3/files
[35]: https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#index-read
[36]: https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#index-IFS
