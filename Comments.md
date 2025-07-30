# Comments

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
# Query Filters

LogScale query filters enable powerful search capabilities through free text matching, field-specific filters, and regular expressions, with each type offering distinct ways to find and filter event data. The documentation covers the syntax and usage of these three filter types, including how multiple filters can be combined with implicit AND operations, along with important considerations for performance and field extraction when using different filtering methods.

Query filters allow you to search LogScale with filters using free text , field matches , and regular expressions .

A filter is a less general kind of expression compared to an expression. Neither is a subset of the other, but Filter is particularly quirky:

Implicit AND is supported in the Filter production so be aware that this:

```
LOGSCALE SYNTAX
foo < 42 + 3
```

means:

```
LOGSCALE SYNTAX
(foo < 42) AND "*+*" AND "*3*"
```

Essentially, two expressions are considered to be logically combined with an AND so that:

```
LOGSCALE
src="client" ip="127.0.0.1"
```

Matches both filter expressions, (for example both a client and localhost address).

The above implies the pipe in the expression, so the above is identical to:

```
LOGSCALE
src="client" | ip="127.0.0.1"
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
LOGSCALE
/regex/
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

---

# Operators

The documentation covers LogScale operators and their usage in comparing field values across strings, numbers, and regular expressions, including detailed explanations of string comparison operators (=, !=, like), numeric operators (&lt;, &gt;, =, etc.), and logical operators (and, or, not). The guide also explains how operators interact with tag fields, provides examples of combining filters with Boolean operators, and demonstrates how to negate filter function expressions for more efficient querying.

Operators allow for comparisons on field values. Comparison operators work on strings, numbers, or regular expressions.

When using operatros:

- The left-hand-side of the operator is interpreted as a field name. If you write 200 = statuscode , LogScale tries to find a field named 200 and test if its value is statuscode . The value must much exactly, including the case.
- For more flexibility with filtering, use the wildcard() function.
- If the specified field is not present in an event, then the comparison always fails unless it is != . You can use this behavior to match events that do not have a given field, using either not (foo = *) or the equivalent foo != * to find events that do not have the field foo .
- To compare two fields, rather than a field and a value, use the test() function. When using test() to care to quote the field and values correction using double quotes to select what is a field and what is a value.

The test() function uses eval expression syntax that is also available in other functions, including eval() , if() , and coalesce() . Also, in the evaluated short hand:



```
LOGSCALE SYNTAX
field := evalExpression.
```


## Comparison Operators on Strings

For string operators, the syntax assumes the value on the right of the operator is a string. For example:

```
LOGSCALE SYNTAX
class like "Bucket"
```

The like operator also supports wildcards, so:

```
LOGSCALE SYNTAX
class like "foo*bar"
```

Will find entries that begin with foo and end with bar .

| Operator   | ▴▾ Case Sensitive  ▴▾   | Description  ▴▾                                                                    |
|------------|-------------------------|------------------------------------------------------------------------------------|
| =          | Yes                     | Field is equal to the entire declared string. Also achieveable using  /regex/ .    |
| !=         | Yes                     | Field does not equal the entire declared string. Also achieveable using  /regex/ . |
| like       | Yes                     | Field is contains the declared string. Also achieveable using  /regex/ .           |

The like operator filters for fields containing the string, but remains case sensitive. The like operator query:

```
LOGSCALE SYNTAX
class like "Bucket"
```

Is therefore equivalent to:

```
LOGSCALE SYNTAX
class like "*Bucket*"
```

Is therefore equivalent to:


```
LOGSCALE SYNTAX
class = *Bucket*
```

Or

```
LOGSCALE SYNTAX
class like /Bucket/
```

Or

```
LOGSCALE SYNTAX
class = /Bucket/
```

## Comparison Operators on Numbers

Numerical operators can be used to filter on a numerical value. The LogScale will attempt to convert the value to a number before comparison, reporting an error if the value cannot be converted. To compare two numerical values, use the test() . For example:

```
LOGSCALE SYNTAX
statuscode = "404"
```

If statuscode is a numeric value, the string will be converted to a number before comparison.

| Query  ▴▾          | Description  ▴▾                                                                                                                                                         |
|--------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| statuscode <  400  | Less than                                                                                                                                                               |
| statuscode <=  400 | Less than or equal to                                                                                                                                                   |
| statuscode =  400  | Equal to                                                                                                                                                                |
| statuscode !=  400 | Not equal to                                                                                                                                                            |
| statuscode >=  400 | Greater than or equal to                                                                                                                                                |
| statuscode >  400  | Greater than                                                                                                                                                            |
| 400 =  statuscode  | The field  400  is equal to  statuscode .                                                                                                                               |
| 400 >  statuscode  | This comparison generates an error. You can only perform a comparison between numbers. In this example,  statuscode  is not a number, and  400  is the name of a field. |

## Filtering on Tag Fields

Tag fields are used to define the datasources for a given event, and have an impact on the storage and performance of queries. For more information, see Datasources . When filtering on a tag field, filters behave in the same way as regular Query Filters . This is recommended to decrease the query time by reducing the amount of data to be searched.

Due to the performance implications, filtering on tag fields should be placed first in the query to query the data. For more information on running queries and selecting fields for filtering, see Multi-line queries.

## Logical Operators

You can combine filters using the and , or , not Boolean operators, and group them with parentheses. ! can also be used as an alternative to unary not .

Examples

| Query  ▴▾                                       | Description  ▴▾                                                                                                                                                                                   |
|-------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| foo and user=bar                                | Match events with  foo  in any field and a  user  field matching  bar .                                                                                                                           |
| foo bar                                         | Since the  and  operator is implicit, you do not need to include it in this simple type of query.                                                                                                 |
| statuscode=404 and  (method=GET or method=POST) | Match events with  404  in their  statuscode  field, and either  GET or  POST  in their  method  field.                                                                                           |
| foo not bar                                     | This query is equivalent to the query  foo  and ( not bar ).                                                                                                                                      |
| !bar                                            | This query is equivalent to the query  not bar .                                                                                                                                                  |
| not foo bar                                     | This query is equivalent to the query (not foo) and bar. This is because the  not  operator has a higher priority than  and  and  or .                                                            |
| foo and not bar or  baz                         | This query is equivalent to the query foo and ((not bar) or baz). This is because LogScale has a defined order of precedence for operators. It evaluates operators from the left to the right.    |
| foo or not bar and  baz                         | This query is equivalent to the query  foo or ((not bar) and  baz) . This is because LogScale has a defined order of precedence for operators. It evaluates operators from the left to the right. |
| foo not  statuscode=200                         | This query is equivalent to the query  foo and statuscode!=200 .                                                                                                                                  |

## Negating the Result of Filter Functions

The not and ! operators can also be used to negate filter function expressions, which is syntactically more clean than passing in an explicit negate=true argument. Examples of this are

```
LOGSCALE SYNTAX
...
| !cidr(ip, subnet="127.0.0/16")
| ...
...
| !in(field, values=[a, b, c])
| ...
...
| !regex("xxx")
| ...
```

---

# Adding Fields

The documentation explains how to add new fields in LogScale through two primary methods: regular expression-based field extraction and function-based field creation using 'as' parameters. Additional topics covered include eval syntax for numeric computations, assignment operators for simplified field creation, and field operators for filtering specific fields with functions, providing users with comprehensive options for field manipulation and data processing.

New fields can be created in two ways:

- Regular Expression-based Field Extraction
- as Parameters

## Regular Expression-based Field Extraction

You can extract new fields from your text data using regular expressions and then test their values. This lets you access data that LogScale did not parse when it indexed the data.

For example, if your log entries contain text such as ... disk\_free=2000 ... , then you can use a query like the following to find the entries that have less than 1000 free disk space.

```
LOGSCALE 
regex("disk_free=(?<space>[0-9]+)") | space < 1000
```

The first line uses regular expressions to extract the value after the equals sign and assign it to the field space , and then filter the events where the extracted field is greater than 1000 .

The named capturing groups ( (?&lt;FIELDNAME&gt;) are used to extract fields in regular expressions. This combines two principles, the usage of grouping in regular expressions using () , and explicit field creation.

The same result can be obtained written using the regex literal syntax:


```
LOGSCALE
@rawstring=/disk_free=(?<space>[0-9]+)/
| space < 1000
```

You can apply repeat to field extraction to yield one event for each match of the regular expression. This allows processing multiple values for a named field, or a field name that matches a pattern, as in this example:

```
LOGSCALE 
regex("value[^=]*=(?<someBar>\\S+)", repeat=true) | groupBy(someBar)
```

On an input event with a field value of:

```
ACCESSLOG 
type=foo value=bar1 valueExtra=bar2 value=bar3
```

the groupBy() sees all three bar values.

```
! Warning In order to use field-extraction this way, the regular expression must be a top-level expression, that is, | between bars | . The following query does not work:
```


```
!! Invalid Example for Demonstration - DO NOT USE 
LOGSCALE 
// DON'T DO THIS - THIS DOES NOT WORK 
type=FOO or /disk_free=(?<space>[0-9]+)/ | space < 1000
```

## @Note

Since regular expressions require additional processing, it is recommended to do as much simple filtering (text or field matching) as possible earlier in the query chain before applying the regular expression function.

## as Parameters

Fields can also be added by functions. Most functions set their result in a field that has the function name prefixed with a \_ by default. For example the count() puts its result in a field \_count .

Most functions that produce fields have a parameter called as . By setting this parameter you can specify the name of the output field, for example:

```
LOGSCALE 
count(as=cnt)
```

Assigns the result of the count() to the field named cnt (instead of the default \_count ).

Many functions can be used to generate new fields using the as argument. For example concat() :

```
LOGSCALE 
concat([aidValue, cidValue], as=checkMe2)
```

Combines the aidValue and cidValue into a single string.

The format() can be used to format information and values into a new value, for example, formatting two fields with a comma separator for use with a table:

```
LOGSCALE SYNTAX 
format(format="%s,%s", field=[a, b], as="combined") | table(combined)
```

See also the Assignment Operator for shorthand syntax for assigning results to a field.

## Eval Syntax

The function eval() can assign fields while doing numeric computations on the input.

The := syntax is short for eval. Use | between assignments.


```
LOGSCALE SYNTAX
... 
| foo := a + b 
| bar := a / b | 
...
```

This is short for the following:

```
LOGSCALE SYNTAX 
... 
| eval(foo = a + b) 
| eval(bar = a / b) |
...
```

## Assignment Operator

You can use the operator := with functions that take an as parameter. When what is on the right hand side of the assignment is a Function Call , the assignment is rewritten to specify the as= argument which, by convention, is the output field name. For example:

```
LOGSCALE SYNTAX 
... 
| foo := min(x) 
| bar := max(x) 
| ...
```

The above is short for this:

```
LOGSCALE SYNTAX 
... 
| min(x, as=foo) 
| max(x, as=bar) 
| ...
```

## Field Operator

The field operator filters a specific field with any function that has the field parameter, so that:

```
LOGSCALE SYNTAX 
... 
| ip_addr =~ cidr(subnet="127.0.0.1/24") 
| ...
```

Is synonymous with:

```
LOGSCALE SYNTAX 
... 
| cidr(subnet="127.0.0.1/24", field=ip_addr) 
| ...
```

This works with many functions, including regex() and replace() .

---

# User Parameters (Variables)

The documentation explains how to implement user-configurable parameters in LogScale queries, allowing dynamic value substitution through user input rather than fixed values. Parameters can be created using a question mark prefix, support default values for automated contexts like triggers and scheduled searches, and offer special syntax for handling multi-value inputs in dashboard implementations.

User-configurable parameters can be added to a query to allow for the user to specify a value in place of a fixed value within the query. The user-configurable value can also be integrated with dashboards and saved searches.

To create a user-supplied parameter, use the ? character in front of the parameter name. For example: ?parameter . The expression can be embedded in the query and will be interpreted by dashboards and saved searches automatically, providing a prompt for userinput:

```
LOGSCALE 
matchstring := ?searchtext
```

In the above example, the named parameter will be searchtext .

For information on using parameters when using Dashboards, see Configuring Dashboard Parameters .

For information on using parameters with saved searches, see User Functions (Saved Searches) .

## Default Parameter Values

For queries that execute in a automated context, for example Alerts or Scheduled Searches , a default value to a parameter can be defined to ensure that the parameter has a value and the query does not fail.

To specify a default value, use the following syntax in your query:

```
LOGSCALE 
?{param=default_value}
```

## Multi-Value Parameters Syntax for Dashboards

When using Multi-value Parameters in dashboards, multiple values can be added at the same time by using commas as a delimiter for user-inputs in the UI. To add multi-value parameters to your query for a dashboard, use the syntax as in the following examples:

| User Input  ▴▾   | Parameter Value Options  ▴▾   |
|------------------|-------------------------------|
| cat, hat         | cat  and  hat                 |
| "cat, hat"       | cat, hat                      |
| \"cat, hat\"     | "cat  and  hat"               |
| \"cat\", \"hat\" | "cat"  and  "hat"             |


---
# Conditional Evaluation

LogScale provides conditional evaluation capabilities through Case Statements and Match Statements, allowing developers to define alternative flows and handle different scenarios in their queries. Case Statements enable multiple test expressions with corresponding results using a semicolon-separated clause structure, while Match Statements offer a switch-like operation that checks conditions against the same field using filters and regular expressions, both supporting default/wildcard clauses for handling unmatched events.

In LogScale the streaming style is not well-suited for procedural-style conditions. However, there are a few ways to do conditional evaluation:

- Case Statements
- Match Statements

## Case Statements

Using case statements, you can define alternative flows in your queries. It is similar to case or cond you might know from many other functional programming languages.

The syntax looks like this:

```
LOGSCALE SYNTAX 
case { 
    pipeline1 
| ... ; 
    pipeline2 
| ... ; 
    pipeline3 
| ... ; 
    * 
| ... 
}
```

Each clause in the pipeline is terminated by a semicolon. For each clause within the case statement, if the expression for this clause returns events, then the clause is considered to have matched.

Typically, each clause takes the form of one or more test expressions followed by one or more results. For example, a single test and result:

```
LOGSCALE 
src="client-side" | type:="client"
```

The first statement tests for a field value, the second creates a new field.

OR for a multiple statement test:

```
LOGSCALE 
src="client-side" 
| ip != "127.0.0.1" 
| type:="local"
```

In this example we are testing both the field src and ip .

The pipeline is considered to have matched if the pipeline returns events. If the test returns true and a value is set, then the clause has matched. If no clauses match, then events are dropped.

You can add a wildcard or default clause that will be used if no other clauses match by using an asterisk in the pipeline:

```
LOGSCALE SYNTAX 
case { ... ; 
* 
| DEFAULT }
```

The default matches all events in the "default case", performing similar to the else part of an if-statement. If you do not add a wildcard clause, any events that do not match any of the explicit clauses will be dropped.

## Conditional Case Example

Let's say we have logs from multiple sources that all have a field named time , and we want to group those hosts by a type identifying what host group to produce percentiles of the time fields, but one for each kind of source.

First, we try to match some text that distinguishes the different types of line. Then we can create a new field type and assign a value that we can use to group by.

```
LOGSCALE 
time=*
| case { 
  src="client-side"
| type := "client";
  src="frontend-server"
| ip != 192.168.1.1
| type := "frontend";
*
| type := "server"
}
| groupBy(type, function=percentile(time))

```

Within the case statement, there are three separate clauses:

```
LOGSCALE SYNTAX 
src="client-side" 
| type := "client";
```

When the src field contains client-side , set the type field to client .

```
LOGSCALE SYNTAX 
src="frontend-server" 
| ip != 192.168.1.1 
| type := "frontend";
```


When the src field contains frontend-server and provided that the ip is not 192.168.1.1 , set the type field to frontend .


```
LOGSCALE SYNTAX
* 
| type := "server"
```

If no other clause matches, set type to server .



## Catch-All Clause

The * clause captures any output that has not matched a previous clause, and can be used either to ignore that information (resulting in no action or output) or to apply a default operation or value. For example, in the sample below the client field is being used to determine whether the IP address is localhost and setting the local field accordingly:

```
LOGSCALE 
case { client = "::1"
| local := "true";
       client = "127.0.0.1"
| local := "true";
       *
| local := "false"}
```

The * clause in this instance sets local to false for any non-matching value.

However, the following is also valid:

```
LOGSCALE 
case { client = "::1"
| local := "true";
       client = "127.0.0.1"
| local := "true";
       * }
```

The above ensures that the non-matching clauses are not processed, but does not create the field unless we have identified a local value.

The following example demonstrates a base match and default response example when looking at the Event\_SimpleName field:

```
LOGSCALE
Event_SimpleName match{
/.*Network.*/ => network_tag := "Network";
* => new_tag := "Other"}
```

## Match Statements

Note: If you are looking for the match() function, see match()

Using match statements, you can describe alternative flows in your queries where the conditions all check the same field. It is similar to the switch operation you might recognize from many other programming languages. The matches on the field support the filters listed in Field Filters and in Regular Expression Filters .

The syntax looks like this:

```
LOGSCALE SYNTAX
field match {
  value => expression
| expression... ;
  /regex/ => expression
| ...;
  * => expression
| ...
}
```

You write a sequence of filter and pipeline clauses to run when the filter matches, separated by a semicolon ( ; ). LogScale will apply each clause from top to bottom until one returns a value (matches the input).

You can use some functions as selectors (in addition to string patterns). More specifically, those functions which test a single field (and do not transform the event).

You can add a wildcard clause match { ... ; * =&gt; * } which matches all events as the "default case", essentially the else of an ifstatement. If you do not add a wildcard clause, then any events that do not match any of the explicit clauses will be dropped. You cannot use the empty clause - you must explicitly write * to match all.

This also means that "*" and * are not equivalent either, and "*" and "**" equivalent.

```
part As opposed to how a regular field test would work - where x=* checks for existence and x=true checks that x matches the string true - in the match (for case statements) the guards * and true are equivalent and matches everything. Because * no longer has the same meaning as in field tests, x match { ** => *} and x match { * => *} are not equivalent since the first checks for existence and the latter accepts everything. are
```

Since, as in case statements, the guards * and true match anything, the existence of a field can be checked using a quote glob "*" or two consecutive globs ** .

The syntax looks like this:

```
LOGSCALE SYNTAX 
field match {
  value => expression
| expression... ;
  /regex/ => expression
| ...;
  in(values=[1, 2, 3]) => expression
| ...; // match if in(field=field, values=[1, 2, 3]) does
  "*" => expression
| ...; // match if the field exists
  * => expression
| ... // match all events
}
```

match statements accept any "pure" Filtering Query Functions (filter functions that do not mutate) that has a field parameter, for example:

```
LOGSCALE SYNTAX 
x match { in(values=[5, 56, 567]) => *;}
```

## Conditional Match Example

Let's say we have logs from multiple sources that all have a field that holds the time spent on some operation, but in different fields and units. We want to get percentiles of the time fields all in the same unit and in one field.

```
LOGSCALE 
logtype match {
    "accesslog" => time:=response_time ;     // Access log is in seconds.
    /server_\d+/ => time:=server_time*1000 ; // These servers log in millis
  }
| groupBy(logtype, function=percentile(time))
```

## Setting a Field's Default Value

You can use the function default() to set the value of a missing or empty field or use the function coalesce() to select the first defined value from a list of expressions.

---





