# DESCRIPTION

    Easily parse and modify riak's app.config from the command line.

# SYNOPSIS

    $ riak-conf [ options ] <command> [arguments]

## OPTIONS

    -c --config <file>  Specify configuration file location to <file>.
    -o --output <file>  Specify save location, or use "-" for stdout.
    -a --all            When listing/searching, show all tuples.
    -n --numbers        Add line numbers to list/search output.
    -f --force          Force, used to overide value type check for modify.
    -d --debug          Set log level to debug.
    -q --quiet          Set log level to suppress usual output.
    --log --loglevel N  Set log level manually to N.
    --diff              Shortcut to display what would be changed.
    --help              Print this help page.
    --man               Prin more verbose help.

## COMMAND OVERVIEW
    
    Use riak-conf help <command> for more details on each
    

    list     List parameters and values
    get      Get value of specific parameter
    add      Add new parameter
    remove   Remove entire parameter
    modify   Change the values of a parameter
    search   List parameters that contain a given string
    help     Display help for commands

## EXAMPLES

    $ riak-conf

        riak_api.pb_ip: "127.0.0.1"
        riak_api.pb_port: 8081
        riak_core.ring_state_dir: "./data/ring"
        ...

    $ riak-conf search http
    
        riak_core.http.[1]: "127.0.0.1" 8091
        riak_kv.http_url_encoding: on

    $ riak-conf add riak_core.http 192.168.0.2 8098
    
        [24/Aug/2012:10:27:23 -0700] INPUT: Coerced 192.168.0.2 into "192.168.0.2"
        [24/Aug/2012:10:27:23 -0700] SAVING: Changes to ../etc/app.config:
        [24/Aug/2012:10:27:23 -0700] DIFF: ---begin diff---
        36c36,37
        <               {http, [ {"127.0.0.1", 8091 } 
        ---
        >               {http, [ {"127.0.0.1", 8091 }, 
        >                       {"192.168.0.2", 8098} 
        [24/Aug/2012:10:27:23 -0700] DIFF: ---end diff---
        [24/Aug/2012:10:27:23 -0700] SAVING: Changes saved.

    $ riak-conf list riak_core.http
        
        riak_core.http.[1]: "127.0.0.1" 8091
        riak_core.http.[2]: "192.168.0.2" 8098

    $ riak-conf riak_core.http.[2] _ 8099

        [24/Aug/2012:10:31:14 -0700] SAVING: Changes to ../etc/app.config:
        [24/Aug/2012:10:31:14 -0700] DIFF: ---begin diff---
        37c37
        <                       {"192.168.0.2", 8098} 
        ---
        >                       {"192.168.0.2", 8099} 
        [24/Aug/2012:10:31:14 -0700] DIFF: ---end diff---
        [24/Aug/2012:10:31:14 -0700] SAVING: Changes saved.

    $ riak-conf remove riak_core.http.[1]

        [24/Aug/2012:10:32:09 -0700] SAVING: Changes to ../etc/app.config:
        [24/Aug/2012:10:32:09 -0700] DIFF: ---begin diff---
        36c36
        <               {http, [ {"127.0.0.1", 8091 }, 
        ---
        >               {http, [  
        [24/Aug/2012:10:32:09 -0700] DIFF: ---end diff---
        [24/Aug/2012:10:32:09 -0700] SAVING: Changes saved.

# COMMANDS

## HELP
    

    $ riak-conf help <command>

        Where <command> is one of:
            list, get, add, remove, modify, search, help

## ADD

    $ riak-conf add <name> <value> [<valueN> ... ]

        Add <value> parameter to <name>, with optional set of <valueN>.
        Use "root" as name to add new base parameter.

## REMOVE

    $ riak-conf remove <name>

        Completely remove the parameter with name <name>.

## MODIFY

    $ riak-conf [modify] <name> <value> [<valueN> ... ]

        Change <name> parameter's value to <value>.
        If parameter is multi-valued, multiple values can be listed at once.
        The underscore (_) can be used to skip over values without changing
        them.

## LIST 

    $ riak-conf list [<name>]
    

        List values of parameters that begin with with <name>, 
        or all if none given.

## GET

    $ riak-conf get <name> [nth] ...

        Return the values of parameter <name>, 
        and optionaly <nth> specific value.
