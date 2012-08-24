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
