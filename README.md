# apache2_mod_authz_dbd
Extension of apache 2.4's **mod_authz_dbd** to include parametrable
parameters for database queries done thru the 'AuthzDBDQuery' directive.

This extension was contributed to the apache server and it's being
made available for other people here while waiting resolution for 
its acceptance.

See https://bz.apache.org/bugzilla/show_bug.cgi?id=58025 for more
details.

## To compile

Checkout the repository, then
 
    cd apache2_mod_authz_dbd
    apxs2  -i -a -c mod_authz_dbd.c -Wc,-Wall -Wc,-g, -Wc,-Wunused

This will compile the module and install it, thus replacing the one
shippied in with apache. Restart apache to use it.

If you're not using the new directive and features, this module
is expected to be backwards compatible with the one shipped with Apache.

## Usage

This module adds a new require directive, 'Require dbd-query', to specify 
any number of parameters we want to feed to 'AuthzDBDQuery'.
These parameters must be space-separated and can be any kind of variables 
supported by Apache as specified in 
https://httpd.apache.org/docs/2.4/expr.html#vars.

The module also extends 'Require dbd-login' and 'dbd-logout' so that they
can use optional parameters if you want to feed them to 'AuthzDBDQuery'.
If you don't use any parameters, you will fallback to the previous
behavior for backwards compatibility.

## Examples:

    <Location "/my/protected/space">
        Require dbd-query %{REQUEST_URI} %{REMOTE_USER} %{REQUEST_METHOD}
        AuthzDBDQuery "SELECT true from uris \ 
                       WHERE u.uris = %s \
                       AND u.user = %s \
                       AND u.request_method = %s \
                       LIMIT 1"
    </Location>

The query parameters will be feed in order and substitute the %s
place holders. Note that you don't need to encapsulate them with quotes
as the query will be compiled. If you feed more query parameters than are
needed by 'AuthzDBDQuery', they will be silently ignored. However, if
you feed less parameters than needed, an error will be logged and
the server will return an 'HTTP 500' status code.

The above query will authorize access to a resource if the database
has an authenticated user's identity and request method record for it.

Here are two examples where you can feed query parameters to 'dbd-login'
and 'dbd-logout'. As stated before, those parameters are optional and
if not used, will use '%{REMOTE_USER}' as a default value, to keep
backwards compatibility.

    <Location "/my/protected/space/login">
        Require dbd-login %{REMOTE_USER}
        AuthzDBDQuery "UPDATE authn SET login = 'true' WHERE user = %s"
    </Location>

    <Location "/my/protected/space/logout">
        Require dbd-logout %{REMOTE_USER}
        AuthzDBDQuery "UPDATE authn SET login = 'false' WHERE user = %s" 
    </Location>
