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
=cut
sub add_tuple {
    my $id = shift;
    my $args = shift;
    my $name = get_name($id);
    my $meta = get_meta_ref($id);
    if ( $$meta->{named_list} ){
        my $list = $$meta->{named_list};
        my $list_meta = get_meta_ref($list);
        my @args = validate_args_for_input(@$args);
        if ( scalar(@args) == 1 ){
            push @args, '[ ]';
        }
        my $tuple = "{".join(", ", @args)."}";
        my @children = get_children($list);
        if ( scalar(@children) > 0 ){
            my $sibling_meta = get_meta_ref($children[-1]);
            my $sibling_end = $$sibling_meta->{end};
            append_to_token(
                $sibling_end,
                ", \n      "."    " x scalar(@b).$tuple
            );
        } else {
            append_to_token(
                $$meta->{named_list},
                "\n      "."    " x scalar(@b).$tuple
            );
        }
    }
}
=head2 REMOVE

    $ riak-conf remove <name>

        Completely remove the parameter with name <name>.
=cut
sub remove_tuple {
    my $id = shift;
    my $name = get_name($id);
    my $child_to_remove = $_[0]->[0];
    my %child_names = map { $tokens[$_][META]->{name} => $_ } get_children($id);
    if ( exists $child_names{$child_to_remove} ){
        set_action_on_tokens($child_names{$child_to_remove}, 'remove');
        $OPT{write}++;
    } else {
        logger(FATAL,'remove',"$child_to_remove not found in $name!");
    }
}

## MODIFY

    $ riak-conf [modify] <name> <value> [<valueN> ... ]

        Change <name> parameter's value to <value>.
        If parameter is multi-valued, multiple values can be listed at once.
        The underscore (_) can be used to skip over values without changing
        them.
=cut
sub modify_tuple { 
    my $id = shift; 
    my @args = @{shift(@_)};
    if ( is_endpoint($id) ){
        my $meta = get_meta_ref($id);
        my @values = @{$$meta->{values}};
        for ( my $arg = 0; $arg <= $#args; $arg++ ){
            next if $args[$arg] eq '_';
            save_arg_to_token($values[$arg], $args[$arg]);
        }
    }
}

## LIST 

    $ riak-conf list [<name>]
    

        List values of parameters that begin with with <name>, 
        or all if none given.
=cut
sub list_tuple {
    if ( $OPT{all} ){
        show_all_tuples(@_);
    } else {
        show_endpoint_tuples(@_);
    }
}

## GET

    $ riak-conf get <name> [nth] ...

        Return the values of parameter <name>, 
        and optionaly <nth> specific value.
=cut
sub get_value {
    my $id = shift;
    my $meta = get_meta_ref($id);
    my @args = @{shift(@_)};
    my @value_ids;
    if ( scalar(@args) > 0 ){
        foreach my $arg (@args){
            if ( $arg && $arg <= scalar(@{$$meta->{values}}) ){
                push @value_ids, $$meta->{values}->[$arg - 1];
            }
        }
    } else {
        @value_ids = @{$$meta->{values}};
    }
    print join("\t", map { $tokens[$_][DATA] } @value_ids)."\n";
}



\# DISPLAY UTILITIES
\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#
sub help {
    my @usage;
    push @usage, '-msg', $\_\[0\] if $\_\[0\];
    my $status = $\_\[0\] ? 1 : 0;
    push @usage, '-exitval', $status;
    pod2usage(@usage);
}

sub command\_help {
    my $cmd = shift;
    my @usage = ( -exitval => 0 );
    if ( exists $commands{$cmd} ){
        $cmd = uc($cmd);
        push @usage, ( -verbose => 99, -sections => \[ "COMMANDS/$cmd" \] );
    } else {
        push @usage, ( -verbose => 99, -sections => \[ "COMMANDS" \] );
    }
    pod2usage(@usage);
}

sub show\_endpoint\_tuples {
    print get\_line\_display($\_\[0\])."\\n" if is\_endpoint($\_\[0\]);
}

sub show\_all\_tuples {
    print get\_line\_display($\_\[0\])."\\n";
}

sub get\_line\_display {
    my $id = shift;
    my $line = "";
    my $meta = get\_meta\_ref($id);
    $line .= sprintf '%5s', $tokens\[$id\]\[LINE\].": " if $OPT{numbers};
    $line .= get\_name($id);
    if ( scalar(@{$$meta->{values}})){
        $line .= ": ".join(" ", map { $tokens\[$\_\]\[DATA\] } @{$$meta->{values}});
    } else {
        $line .= ".\*";
    }
    return $line;
}

sub logger {
    my $type = shift;
    my $context = shift;
    my $error = shift;
    if ( $type >= FATAL ){
        if ( $context eq 'input' ){
            help($error);
        } else {
            die(log\_line($context, $error));
        }
    } elsif ( $type == WARN ){
        warn log\_line($context, $error) if WARN >= $OPT{log};
    } else {
        print log\_line($context, $error) if $type >= $OPT{log};
    }
}

sub log\_line {
    my $context = shift;
    my $error = shift;
    my $line = "\[".strftime("%d/%b/%Y:%H:%M:%S %z", localtime(time()))."\] ";
    $line .= uc($context).": $error\\n";
}

\# DATA ACCESS FUNCTIONS
\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

sub check\_if\_named\_list {
    my $id = shift;
    my $meta = get\_meta\_ref($id);
    if ( scalar(@{$$meta->{children}}) == 1 &&
         $tokens\[$$meta->{children}->\[0\]\]\[META\]->{type} eq 'list' ){

        $tokens[$id][META]->{named_list} = $$meta->{children}->[0];
    }
}

sub get\_meta\_ref {
    my $id = defined $\_\[0\] ? shift(@\_) : $\#tokens;
    if ( my $meta = shift ){
        return \\$tokens\[$id\]\[META\]->{$meta};
    } else {
        return \\$tokens\[$id\]\[META\];
    }
}

sub is\_endpoint{
    return scalar(@{$tokens\[$\_\[0\]\]\[META\]->{values}}) > 0;
}

sub get\_name {
    my $id = shift;
    my $meta = get\_meta\_ref($id);
    my @name;
    foreach my $b\_id (@{$$meta->{branch}}){
        next if $b\_id == 0;
        my $b\_meta = get\_meta\_ref($b\_id);
        push @name, $$b\_meta->{name} if $$b\_meta->{type} eq 'tuple';
    }
    push @name, $$meta->{name} || "\[\]";
    return join(".", @name);
}

sub find\_last {
    return find\_last\_from($\#tokens, @\_);
}

sub find\_last\_from {
    my $id = shift;
    my %query = map { $\_ => 1 } @\_;
    until ( exists $query{$tokens\[$id\]\[TYPE\]} || $id < 0 ) { $id-- }
    return $id;
}

sub get\_children {
    my $id = shift;
    my $meta = get\_meta\_ref($id);
    my @children;
    foreach my $child (@{$$meta->{children}}){
        my $child\_meta = get\_meta\_ref($child);
        if ( $$child\_meta->{type} eq 'list' ){
            push @children, get\_children($child);
        } else {
            push @children, $child;
        }
    }
    return @children;
}



\# DATA MODIFICATION FUNCTIONS
\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#\#

sub set\_action\_on\_tokens {
    my $id = shift;
    my $action = shift;
    my $meta = get\_meta\_ref($id);
    set\_action($id, $action);
    if ( exists $$meta->{end} ){
        my $start = $id + 1;
        foreach my $part ($start..$$meta->{end}){
            set\_action($part, $action);
        }
        if ( exists $$meta->{comma} ){ 
            set\_action($$meta->{comma}, $action);
        } else {
            if ( $$meta->{order} > 1 ){
                my $parent\_meta = get\_meta\_ref($$meta->{parent});
                my $older\_sibling = get\_meta\_ref(
                    $$parent\_meta->{children}\[$$meta->{order} - 2\] \# b/c of 0idx
                );
                if ( exists $$older\_sibling->{comma} ){
                    set\_action($$older\_sibling->{comma}, $action);
                } else {
                    logger(WARN,'syntax',
                        "Older sibbling of $id doesn't have a comma:");
                }
            }
        }
    }
    



}

sub set\_action { $tokens\[$\_\[0\]\]\[META\]->{action} = $\_\[1\]; }

sub save\_arg\_to\_token {
    my $id = shift;
    my $arg = shift;
    my $type = $tokens\[$id\]\[TYPE\];
    my $is\_valid = 0;
    if ( is\_of\_type($arg, $type) ){
        $is\_valid = 1;
    } elsif ( $type eq 'text' ){
        $arg = "\\"$arg\\"";
        if ( is\_of\_type($arg, $type) ){
            $is\_valid = 1;
        }
    }

    if ( $is_valid || $OPT{force} ){
        modify_token_content($id, $arg);
    } else {
        warn "Argument $arg does not match expected type of $type\n";
        die "Use -force to override\n" unless $OPT{force};
    }
}

sub modify\_token\_content {
    my $id = shift;
    my $content = shift;
    $tokens\[$id\]\[0\] = $content; \#BOOM
    $OPT{write}++;
}

sub get\_token\_stream {
    my $start = shift;
    my $end = shift || $\#tokens;
    my $offset = $tokens\[$start\]\[POS\];
    my $return = "";
    foreach my $token (@tokens\[$start..$end\]){

        if ( exists $token->[META]->{action} ){
            next if $token->[META]->{action} eq 'remove';
        } else {
            $return .= $token->[DATA];
        }
    }
    return $return;
}

sub append\_to\_token {
    my $id = shift;
    my $append = shift;
    $tokens\[$id\]\[0\] .= $append; \#BOOM
    $OPT{write}++;
}
