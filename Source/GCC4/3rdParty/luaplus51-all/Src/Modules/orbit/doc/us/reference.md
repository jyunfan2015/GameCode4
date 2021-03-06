## Reference Manual

This is a short reference of all of Orbit's (and Orbit apps) methods.

## Module `orbit`

**orbit.new([*app*])** - creates a new Orbit application, returning it (as a module).
If *app* is a string this is the application's name, and sets field \_NAME. If *app*
is a table it is used instead of a fresh table. This means you can pass `orbit.new`
to the `module` function

**orbit.htmlify(*func*[, *func*...])** - modifies the environment of *func* to
include functions that generate HTML

## Orbit apps

**app.run(*wsapi\_env*)** - WSAPI entry point for application, generated by Orbit

**app.real\_path** - the root of the application in the filesystem, defaults to
the path inferred by WSAPI launchers (`wsapi.app\_path`), but can be overriden

**app.mapper** - mapper used by `app:model`, defaults to an instance of `orbit.model`,
but can be overriden

**app.not\_found** - default handler, sends a 404 response to the client, can be overriden.
The handler receives a `web` object

**app.server\_error** - error handler, sends a 500 response with stack trace to the
client, can be overriden. The handler receives a `web` object

**app:dispatch\_get(*func*, *patt*[, *patt*...])** - installs *func* as a GET handler for
the patterns *patt*. *func* receives a `web` object and the captures; this handler should return
the contents of the response or **app.reparse** if you want to hand control back to
the dispatcher and let it continue matching;

**app:dispatch\_post(*func*, *patt*[, *patt*...])** - installs *func* as a POST handler for
the patterns *patt*. *func* receives a `web` object and the captures; this handler should return
the contents of the response or **app.reparse** if you want to hand control back to
the dispatcher and let it continue matching;

**app:dispatch\_put(*func*, *patt*[, *patt*...])** - installs *func* as a PUT handler for
the patterns *patt*. *func* receives a `web` object and the captures; this handler should return
the contents of the response or **app.reparse** if you want to hand control back to
the dispatcher and let it continue matching;

**app:dispatch\_delete(*func*, *patt*[, *patt*...])** - installs *func* as a DELETE handler for
the patterns *patt*. *func* receives a `web` object and the captures; this handler should return
the contents of the response or **app.reparse** if you want to hand control back to
the dispatcher and let it continue matching;

**app:dispatch\_wsapi(*func*, *patt*[, *patt*...])** - installs *func* as a WSAPI handler for
the patterns *patt*. *func* receives the unmodified WSAPI environment

**app:dispatch\_static(*patt*[, *patt*...])** - installs a static files handler for
the patterns *patt*. This handler assumes the PATH\_INFO is a file relative to
`app.real_path` and sends it to the client. The MIME type is detected from
the extension (default application/octec-stream)

**app:serve\_static(*web*, *filename*)** - returns the content of file *filename* (which can be
anywher in the filesystem), while setting the appropriate headers with the file's MIME
type

**app:htmlify(*patt*[, *patt*...])** - same as `orbit.htmlify`, but changes all of the
app's module functions that match the patterns *patt*

**app:model(...)** - calls `app.mapper:new(...)`, so the behavior depends on the mapper
you are using

## `web` methods

The *web* objects inherit the functions of module `wsapi.util` as methods.

**web.status** - status to be sent to server (default: "200 Ok")

**web.headers** - headers to be sent to server, a Lua table (default has Content-Type as text/html)

**web.response** - body to send to the server (default is blank)

**web.vars** - original WSAPI environment

**web.prefix** - app's prefix (if set in the app's module) or SCRIPT\_NAME

**web.suffix** - app's suffix (if set in the app's module)

**web.real\_path** - location of app in filesystem, taken from wsapi\_env.APP\_PATH, or app's module real\_path, or "."
 
**web.doc\_root, web.path\_info, web.script\_name, web.path\_translated, web.method** - server's document root, PATH\_INFO,
SCRIPT\_NAME, PATH\_TRANSLATED, and REQUEST\_METHOD (converted to lowercase)

**web.GET, web.POST** - GET and POST variables

**web.input** - union of web.GET and web.POST

**web.cookies** - cookies sent by the browser

**web:set\_cookie(*name*, *value*)** - sets a cookie to be sent back to browser

**web:delete\_cookie(*name*)** - deletes a cookie

**web:redirect(*url*)** - sets status and headers to redirect to *url*

**web:link(*part*, [*params*])** - creates intra-app link, using web.prefix and web.suffix, and encoding *params*
as a query string

**web:static\_link(*part*)** - if app's entry point is a script, instead of path, creates a link to the app's vpath
(e.g. if app.prefix is /foo/app.ws creates a link in /foo/*part*), otherwise same as web:link

**web:empty(*s*)** - returns true if *s* is nil or an empty string (zero or more spaces)

**web:content\_type(*s*)** - sets the content type of the reponse to *s*

**web:empty\_param(*name*)** - returns true if input parameter *name* is empty (as web:empty)

**web:page(*name*, [*env*])** - loads and render Orbit page called *name*. If *name* starts with / it's relative to
the document root, otherwise it's relative to the app's path. Returns rendered page. *env* is an optional environment
with extra variables.

**web:page_inline(*contents*, [*env*])** - renders an inline Orbit page

## Module `orbit.cache`

**orbit.cache.new(*app*, [*base\_path*])** - creates a page cache for *app*, either in memory or at the filesystem
path *base\_path* (**not** relative to the app's path!), returns the cache object

**a\_cache(*handler*)** - caches *handler*, returning a new handler; uses the PATH\_INFO as a key to the cache

**a\_cache:get(*key*)** - gets value stored in *key*; usually not used, use the previous function instead

**a\_cache:set(*key*, *val*)** - stores a value in the cache; use a\_cache(*handler*) to encapsulate this behavior

**a\_cache:invalidate(*key*)** - invalidates a cache value

**a\_cache:nuke()** - clears the cache

## Module `orbit.model`

**orbit.model.new([*table\_prefix*], [*conn*], [*driver*], [*logging*])** - creates a new ORM mapper. *table\_prefix* (default "")
is a string added to the start of model names to get table names; *conn* is the database connection (can be set
later); *driver* is the kind of database (currently "sqlite3", the default, and "mysql");  *logging* 
sets whether you want logging of all queries to `io.stderr`. Returns a mapper instance,
and all the parameters can be set after this instance is created (via a\_mapper.table\_prefix, a\_mapper.conn, 
a\_mapper.driver, and a\_mapper.logging)

**orbit.model.recycle(*conn\_builder*, *timeout*)** - creates a connection using *conn\_builder*, a function
that takes no arguments, and wraps it so a new connection is automatically reopened every *timeout* seconds

**a\_mapper:new(*name*, [*tab*])** - creates a new model object; *name* is used together with a\_mapper.table\_prefix to
form the DB table's name; fields and types are instrospected from the table. *tab* is an optional table that
is used as the basis for the model object if present

**a\_model.model** - the mapper for this model

**a\_model.name, a\_model.table\_name** - the name of the model and its backing table

**a\_model.driver** - the DB driver used

**a\_model.meta** - metainformation about the model, instorspected from the table

**a\_model:find(*id*)** - finds and returns the instance of the model with the passed *id* (keyed using
the `id` column of the table (must be numeric)

**a\_model:find\_first(*condition*, *args*)** - finds and returns the first instance of the model that
matches *condition*; *args* can determine the order (args.order), specify which fields should be returned
(args.fields, default is all of them), and inject fields from other tables
(args.inject)

Example: `books:find_first("author = ? and year_pub > ?", { "John Doe", 1995, order = "year_pub asc" })`

**a\_model:find\_all(*condition*, *args*)** - finds and returns all instances of the model that
matches *condition*; *args* can determine the order (args.order), specify which fields should be returned
(args.fields, default is all of them), limit the number of returned rows (args.count), return only
distinct rows (args.distinct), and inject fields from other tables (args.inject)

Example: `books:find_first("author = ? and year_pub > ?", { "John Doe", 1995, order = "year_pub asc", count = 5, fields = { "id", "title" } })`

**a\_model:new([*tab*])** - creates a fresh instance of the model, optionally using *tab* as initial
values

**a\_model:find\_by\_xxx(*args*)** - finds and returns the first instance of the model building the
condition from the method name, a la Rails' ActiveRecord; *args* works the same way as in **find\_first**, above

**a\_model:find\_all\_by\_xxx(*args*)** - finds and returns all instances of the model building the
condition from the method name, a la Rails' ActiveRecord; *args* works the same way as in **find\_all**, above

Example: `books:find_all_by_author_or_author{ "John Doe", "Jane Doe", order = "year_pub asc" }`

**an\_instance:save([*force\_insert*])** - saves an instance to the DB, commiting changes or creating the backing record if
the instance is new; if *force\_insert* is true always do an insert instead of an update

If there's a row called `created_at` this row is set to the creation date of the record; if there's a row
called `updated_at` this row is set to the last update's date.

**an\_instance:delete()** - deletes an instance from the DB

## Module `orbit.routes`

This module has a single function, **R**, and you can also call the module itself. This function takes a PATH\_INFO pattern where the path segments (the parts of the path separated by `/` or `.`) can be named parameters (`:name`), *splats* (\*), optional parameters (`?:name?`), or literals, and returns a pattern.

A pattern is an object with two methods:

**patt:match(*str*)** - tries to match *str* and returns a hash of named parameters; splats are returned in a `splat` array inside this hash.

**patt:build(*params*)** - builds the original pattern string from a hash of named parameters

Some examples of patterns, what they match, and the parameter hashes:

* '/foo', matches '/foo', empty hash
* '/foo/:bar/baz' matches '/foo/orbit/baz' or '/foo/xxx.yyy/baz' with hashes { bar = 'orbit' } and { bar = 'xxx.yyy' }
* '/:name.:ext' matches '/xxx.yyy' or '/filename.ext' with hashes { name = 'xxx', ext = 'yyy' } and { name = 'filename', ext = 'ext' }
* '/*/foo/*' matches '/bar/foo' or '/bar/foo/bling/baz/boom' with hashes { splat = { 'bar' } } and { splat = { 'bar', 'bling/baz/boom' } }
* '/?:foo?/?:bar?' matches '/' or '/orbit' or '/orbit/routes' with hashes {} and { foo = 'orbit' } and { foo = 'orbit', bar = 'routes' }

And several more, see `test/test_routes.lua` for a comprehensive test suite.
