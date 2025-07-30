## Comments

The CrowdStrike Query Language (CQL) supports both single-line and multi-line commenting capabilities for code documentation. Single-line comments using // are ideal for end-of-line annotations, while multi-line comments using /* */ allow for more detailed search descriptions and comprehensive documentation within queries.

The CrowdStrike Query Language (CQL) supports // single-line and /* multi-line */ comments.

Single-line comments should be used at the end of a line, for example:

```xql
#host=github #parser=json 
| // Search for host and parser
repo.name=docker/*
| groupBy(repo.name, function=count()) 
| sort()
```

Multi-line comments are useful to provide a deeper description or documentation for a search. For example:

```xql
/* Search for killed processes
   Set the <signal> type and <process> name */
?{signal="*" }
| ?{process="*"}
| /Service exited due to (?<signal>\S+)/
| signal = ?signal
| /sent by (?<process>\S+)\[\d+\]/
| process = ?process
```
---
## Query Filters

LogScale query filters enable powerful search capabilities through free text matching, field-specific filters, and regular expressions, with each type offering distinct ways to find and filter event data. The documentation covers the syntax and usage of these three filter types, including how multiple filters can be combined with implicit AND operations, along with important considerations for performance and field extraction when using different filtering methods.

Query filters allow you to search LogScale with filters using free text , field matches , and regular expressions .

A filter is a less general kind of expression compared to an expression. Neither is a subset of the other, but Filter is particularly quirky:

Implicit AND is supported in the Filter production so be aware that this:

```
LOGSCALE SYNTAX foo < 42 + 3
```

means:

```
LOGSCALE SYNTAX (foo < 42) AND "*+*" AND "*3*"
```

Essentially, two expressions are considered to be logically combined with an AND so that:

```
LOGSCALE src="client" ip="127.0.0.1"
```

Matches both filter expressions, (for example both a client and localhost address).

The above implies the pipe in the expression, so the above is identical to:

```
LOGSCALE src="client" | ip="127.0.0.1"
```

## Free-Text Filters

The most basic query in LogScale is to search for a particular string in any field of events. All fields (except for the special @id , @timestamp , @ingesttimestamp fields and the tag fields ) are searched, including @rawstring .

Free-text filters operate on the fields in the events that are present at the start of the pipeline when performing a search. Free-text search does not take into account any fields added or removed within the pipeline.

When a free-text search is applied in a parser this differs; the event is processed as it is present at the point where the free-text search occurs. LogScale recommends using Field Filters whenever possible within a parser to avoid ambiguous matches.

Free-text search does not specify the order in which fields are searched. When not extracting fields, the order in which fields are checked is not relevant as any match will let the event pass the filter.

But when extracting fields using a regular expression, matches can yield non-deterministic extracted fields. To make extracted fields be the same if a match was also possible in the older versions, LogScale prefers a match on @rawstring before trying other fields when extracting fields.

```
NOTE:
You can perform more complex regular expression searches on all fields of an event by using the regex() function or the /regex/ regex syntax.
```

| Query  ▴▾          | Description  ▴▾                                                                                                                                                                                                     |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| foo                | Find all events matching  foo  in any field of the events. Without a field specification, the text is treated as if it has a leading and trailing wildcard. For example  *foo*                                      |
| "foo bar"          | Use quotes if the search string contains white spaces or special characters, or is a keyword. Without a field specification, the text is treated as if it has a leading and trailing wildcard. For example  "*foo*" |
| "msg:\"welcome\" " | You can include quotes in the search string by escaping them with backslashes.                                                                                                                                      |

You can also use a regular expression on all fields. To do this, write the regular expression, for example /regex/ :

| Query   |  Description                                               |
|---------|------------------------------------------------------------|
| /foo/   | Find all events matching  foo  in any field of the events. |
| /foo/i  | Find all events matching  foo  ignoring case.              |

## Field Filters

Besides the free-text filters , you can also query specific event fields, both as text and as numbers.

| Query  ▴▾              | Description  ▴▾                                                                  |
|------------------------|----------------------------------------------------------------------------------|
| url = *login*          | The  url  field contains  login . You can use  *  as a wild card.                |
| user = *Turing         | The  user  field ends with Turing.                                               |
| user = "Alan  Turing"  | The  user  field equals Alan Turing.                                             |
| user != "Alan  Turing" | The  user  field does not equal Alan Turing.                                     |
| url != *login*         | The url field does not contain  login .                                          |
| user = *               | Match events that have the field  user .                                         |
| user != *              | Match events that do not have the field  user .                                  |
| name = ""              | Match events that have a field called  name  but with the empty string as value. |
| user= "Alan Turing"    | You do not need to put spaces around operators (for example,  =  or != ).        |

## Regular Expression Filters

In addition to globbing ( * appearing in match strings) you can match fields using regular expressions.

To use a regular expression, enclose the regular expression in two forward slashes. For example:

```
LOGSCALE /regex/
```

The use of a regex in this syntax is similar to using the regex() . There are some differences between /regex/ and regex() operations:

- /regex/ searches:
- @rawstring

- All extracted/parsed fields
- field = /regex/ searches:
- Only the specified field using the qualified regular expression
- regex() searches:
- a single field (default is @rawstring )

```
!! Important The different regex syntax operations are significantly different in performance: the /regex/ searching over all fields is slower compared to using either regex() or /regex/ that search on a single, specific field instead.
```



| Query                                                    | Description                                                                                                                                     |
|-----------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| url = /login/                                                   | The  url  field contains  login .                                                                                                                     |
| user = /Turing$/                                                | The  user  field ends with  Turing .                                                                                                                  |
| loglevel =  /error/i                                            | The  loglevel  field matches  error  case insensitively; for example, it could be  Error ,  ERROR  or  error .                                        |
| /user with id (? <id>\S+) logged                 in/  | top(id) | The user id is extracted into a field named  id  at search time. The  id  is then used in the top function to find the users that logged in the most. |

Certain functions can also be used to filter events. For a list of these functions, see Filtering Query Functions .
---





