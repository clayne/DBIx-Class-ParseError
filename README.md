# NAME

DBIx::Class::ParseError - Extensible database error handler

# SYNOPSIS

From:

    DBIx::Class::Storage::DBI::_dbh_execute(): DBI Exception: DBD::mysql::st execute failed: Duplicate entry \'1\' for key \'PRIMARY\' [for Statement "INSERT INTO foo ( bar_id, id, is_foo, name) VALUES ( ?, ?, ?, ? )" with ParamValues: 0=1, 1=1, 2=1, 3=\'Foo1571434801\'] at ...

To:

    use Data::Dumper;
    my $parser = DBIx::Class::ParseError->new(schema => $dbic_schema);
    print Dumper( $parser->process($error) );

    # bless({
    #    'table' => 'foo',
    #    'columns' => [
    #        'id'
    #    ],
    #    'message' => 'DBIx::Class::Storage::DBI::_dbh_execute(): DBI Exception: DBD::mysql::st execute failed: Duplicate entry \'1\' for key \'PRIMARY\' [for Statement "INSERT INTO foo ( bar_id, id, is_foo, name) VALUES ( ?, ?, ?, ? )" with ParamValues: 0=1, 1=1, 2=1, 3=\'Foo1571434801\'] at ...',
    #    'operation' => 'insert',
    #    'column_data' => {
    #        'name' => 'Foo1571434801',
    #        'bar_id' => '1',
    #        'id' => '1',
    #        'is_foo' => '1'
    #    },
    #    'source_name' => 'Foo',
    #    'type' => 'primary_key'
    # }, 'DBIx::Class::ParseError::Error' );

# DESCRIPTION

This a tool to extend DB errors from [DBIx::Class](https://metacpan.org/pod/DBIx::Class) (basically, database error
strings wrapped into a [DBIx::Class::Exception](https://metacpan.org/pod/DBIx::Class::Exception) obj) into an API to provide
useful details of the error, allowing app's business layer or helper scripts
interfacing with database models to instrospect and better handle errors from
multiple DBMS.

## ERROR CASES

This is a non-exausted list of common errors which should be handled by this
tool:

- primary key
- foreign key(s)
- unique key(s)
- not null column(s)
- data type
- missing column
- missing table

# CUSTOM ERRORS

You may find your code throwing exceptions that you would like to generate custom
errors for. You can specify them in the constructor:

    my $parser = DBIx::Class::ParseError->new(
        schema => $dbic_schema,
        custom_errors => {
            locking_failed => qr/Could not update due to version mismatch/i
        }
    );

The `custom_errors` key must point to a hash references whose values are
regular expressions to match against the error. Due to the unpredictable
nature of these errors, the exception will like not have additional
information beyond the error message and the error message.

The parser will attempt to match custom errors before standard errors. Any
error will have the string `custom_` prepended, so the above error will be
reported as `custom_locking_failed`.

# DRIVERS

Initial fully support for errors from the following DBMS:

- SQLite

    See [DBIx::Class::ParseError::Parser::SQLite](https://metacpan.org/pod/DBIx::Class::ParseError::Parser::SQLite).

- MySQL

    See [DBIx::Class::ParseError::Parser::MySQL](https://metacpan.org/pod/DBIx::Class::ParseError::Parser::MySQL).

- PostgreSQL

    See [DBIx::Class::ParseError::Parser::PostgreSQL](https://metacpan.org/pod/DBIx::Class::ParseError::Parser::PostgreSQL).

# AUTHOR

wreis - Wallace reis <wreis@cpan.org>

# CONTRIBUTORS

Ovid - Curtis "Ovid" Poe <ovid@cpan.org>

# COPYRIGHT

Copyright (c) the ["AUTHOR"](#author).

# LICENSE

This library is free software and may be distributed under the same terms
as perl itself.
