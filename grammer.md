# Comments

The CrowdStrike Query Language (CQL) supports both single-line and multi-line commenting capabilities for code documentation. Single-line comments using // are ideal for end-of-line annotations, while multi-line comments using /* */ allow for more detailed search descriptions and comprehensive documentation within queries.

The CrowdStrike Query Language (CQL) supports // single-line and /* multi-line */ comments.

Single-line comments should be used at the end of a line, for example:

```
#host=github #parser=json 
| // Search for host and parser
repo.name=docker/*
| groupBy(repo.name, function=count()) 
| sort()
```

Multi-line comments are useful to provide a deeper description or documentation for a search. For example:

```
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

# Array Syntax

The documentation explains array syntax in LogScale, covering both flat and nested arrays, including how arrays are structured, accessed, and manipulated using JSON-like notation with square brackets and dots. It details structured array syntax for working with arrays of objects, array notation rules for field naming, and provides comprehensive examples of array operations using functions like array:contains(), array:eval(), and objectArray:eval().

Falcon LogScale operates on both flat arrays and nested arrays. Within LogScale there is no distinct array type, but LogScale is able to operate on array-like objects using syntax similar to manipulating JSON arrays and objects. Within LogScale, an array is an ordered collection of values: each value is called an element, and each element has a numeric position in the array, known as its index. Nested arrays are fields that consist of an array where each element is another array or object, also known as arrays of objects or arrays of arrays.

Arrays can be parsed from incoming events using the splitString() or parseJson() functions.

As a part of the CrowdStrike Query Language, there is syntax that applies to array functions, for example array:contains() , array:eval() and objectArray:eval() that we refer to as array syntax. Some of the array functions are used on flat arrays, some on nested arrays. The objectArray:eval() function operates on arrays of objects, the nested arrays - note that it can only read the structured array fields, it cannot write to them.

The array functions operate on fields that follow this specific naming conventions syntax.

The array syntax is similar to the one used by JSON, where [ and ] are used for indexing within arrays and . for selecting members in objects. For example:

```
JSON 
["jsmith","tmcdonald"]
```

Is an array with two elements, the first element username[0] has the value jsmith .

While the object:



```
JSON
{username: "jsmith", name: "Jon Smith"} 
```


The username can be extracted using user.username.
The following table compares the JSON and the parsed LogScale form of the same array and object structures. The final column shows the syntax to use when referring to the whole array.

```
| JSON                                                  | Parsed LogScale event                                      | array name syntax       |
|-------------------------------------------------------|------------------------------------------------------------|-------------------------|
| ["jsmith", "tmcdonald"]                               | username[0]: "jsmith"  username[1]: "tmcdonald"            | username[]              |
| [ {"username": "jsmith", "name": "Jon Smith"}, {"username": "jdoe", "name": "Jane Doe"}   ] | users[0].user.username: "jsmith"  users[0].user.name: "Jon Smith"  users[1].user.username: "jdoe" users[1].user.name: "Jane Doe" | users[]                 |
| [["a", "b"], ["c", "d"]]                              | foo[0][0]: "a" foo[0][1]: "b" foo[1][0]: "c" foo[1][1]: "d" | foo[0][], and foo[1][]  |

```

## Structured Array Syntax

Let's start with an example usage:

```
objectArray:eval("in[]", asArray="out[]", function={out := format("%s:%s", field=[in.key, in.value])})
input:
       in[0].key = x
       in[0].value = y
       in[1].key = a
       in[1].value = b
output:
       out[0] = x:y
       out[1] = a:b
```

objectArray:eval("in[]", asArray="out[]", function=function) iterates over the array from start to end (or to the first "hole" in the array).

For each array entry, the given function is applied.

For each entry, at index i, a field out[i] is created if the function writes a value to the output array name field ("out" in this example).

Within the function, a special pattern to access sub-selections of array entries is supported.

This pattern is:

in (the input array) followed by:

- .&lt;subselection&gt;
- or [&lt;number&gt;]
- or any combination of the above

## Examples:

- in.key
- in.others[0].foo
- in[0][1]

This pattern is what allows the function to operate on arrays of objects, and arrays of arrays, and other arrays of structured data.

Semantically, given the input array "in", an array index i, and an access in.&lt;subselection&gt; this will be translated to the field name in[i].&lt;subselection&gt; . Similarly, in[2] is translated to in[i][2] .

Note: The character sequence could contain another dot. For example in.foo.bar becomes in[i].foo.bar .

Note: At present, it is only possible to read structured array fields. Writing to them is unsupported, but is a possible future extension. For that reason, at present we disallow modifying the existing array.

The var parameter can be used to give a different name to the input array variable inside the function argument. This is particularly useful whenever the input array name is very long. Example:

```
objectArray:eval("someVeryLongName[]", asArray="out[]", var=x, function={out := format("%s:%s", field= [x.key, x.value])})
```

## Syntax Array Notation

Fields with names consisting of a valid array-prefix followed by an array entry are treated as entries of an array. The indices must be continuously increasing from 0 and upwards.

The entire array is referenced within array function using the array-prefix followed by [] that indicates the entire range of entries, for example username[] .

Simplified, the valid array-prefix is a sequence of members and array entries. More formally, a valid array-prefix is either:

- an identifier - that is, a sequence of alphanumerical characters that starts with a letter, an underscore ( \_ ), @ , or # .
- If the member name is not an identifier, for example because it starts with a number or contains other special characters, the name needs to be quoted using ` like this: user.`e-mail`[] .
- a valid array-prefix followed by a member operator and an identifier - the member operator is a .
- or a valid array-prefix followed by an array entry - the array entry is an index surrounded by square brackets.

Examples of field names representing entries in an array, and the reference to that array:

| Field names following the array syntax  ▴▾   | How to reference in array functions  ▴▾   |
|----------------------------------------------|-------------------------------------------|
| a[0], a[1], a[2]                             | a[]                                       |
| a.b[0], a.b[1], a.b[2]                       | a.b[]                                     |
| a[0][0], a[0][1]                             | a[0][]                                    |
| user.e-mail[0], user.e-mail[1]               | user.`e-mail`[]                           |

```
! Warning

If the array is a structured array, for example, a[][0] or a[].b , use the objectArray: functions.

For arrays, use the array: functions.
```
---
# Expressions

The documentation explains LogScale's expression language capabilities, including basic values, operators, and functions that allow users to perform computations and comparisons with strings, numeric values, and boolean data. The content covers essential expression components like literal values, field references, comparison operators, numerical operations, and specialized functions such as if(), coalesce(), and text:contains(), while emphasizing that LogScale treats all values as strings that are interpreted based on their context.

Some LogScale functions and constructs allow writing expressions instead of simple values or field names, for example, to perform computations. The most common places where expressions are used are the test() function and the eval() function and the syntactic short-hand field := expression , but there are other functions that take expressions as arguments, like coalesce() , if() , getField() , and text:contains() . For more information about syntax rules of expressions, see also Grammar Subset .

## Basic Expressions and Values

LogScale's expression language can be used to compute and compare strings that represent text, numeric, and boolean values. Values can be stored in fields on the current event or used as arguments to functions that support expressions, like text:contains() or if() .

It is important to keep in mind that in LogScale there are no types associated with values at runtime, values are basically strings that are interpreted in the context they are used. For example, the query fragment a := "2" | b := 4 /

lower(a) results in b having the value 2 , and a := "2" | b := format("%s%s", field=[a, a]) + 1 results in b having the value 23 . , which

The basic expressions are literal strings, like "some test" , literal booleans like false , and literal numbers, like 1 , 0.5 evaluate to that value.

The values of fields in the event can be referred to by the field name, for example field . If the event does not contain a field of that name, the expression produces the special value null. This null value is handled specially; for example, the expression field &gt; 3 , where the field does not exist, produces a null value, and assigning the null value to a new field in an statement like result := field &gt;3  will not create (or overwrite) the field.

```
@Note they are technically just strings, are interpreted as boolean values in expression contexts, The strings "true" and "false" , as well as the bare words true and false , while for example when using the function if() .
```



## Operators

Expressions can be combined using comparison, numerical and logical operators, and put into parentheses. Using operators with values of the wrong type, for example subtracting strings, does not create a compile-time error, but usually produces a null value at runtime.

The numerical comparison operators are &lt; , &gt; , &lt;= and &gt;= compare numerical values and produce either true or false . If the operands are not numbers, no value is produced:
```
"a" &gt; 3 Comparing a string to a number, result is null. 
```
```
1 &gt; 3 Comparing two numbers, result is false. 
```
```
field &gt; 3 Comparing the value in a field to a number, result is true/false if the field exists and has a numeric value, null otherwise.
```

The numerical operators + , -, / , * , % take one or two values and compute the corresponding numerical operation if both operands are numbers; otherwise, a null value is produced:
```
"a" / 2 Dividing a string and a number, result is null. 
```
```
4 / 2 Dividing two numbers, result is is a number. 
```
```
field / 2 Dividing the value of the field to a number, result is a number if the field exists and has a numeric value, null otherwise.
```
The general comparison operators == and != compare two values and produce true or false , if both values are non-null:

| Operators   | Details   |
|---------------------|-----------------------------------------------------------------------------------------------------------|
| "a" == 2            | Comparing two values of different types, result is false.                                                 |
| 1 != 2              | Comparing two numbers, result is true.                                                                    |
| field ==  2         | Comparing the value of the field to a number, result is a true/false if the field exists, null otherwise. |


Logical negation ! , computes the logical negation of a boolean value, or whether a numerical value is equal to 0 . If the value is neither a boolean nor a numeric value, the result is null:

| Operators   | Details   |
|------------------|--------------------------------------------------------------------------------------------------------------------------------|
| !false,  !true   | Negation of literal boolean value, result is true and false, respectively.                                                     |
| !field           | Negation of value in field field, result is true if the field contains a numeric value or true / false , otherwise it is null. |
| !0, !1           | Negation of numeric values, result is true and false, respectively                                                             |
There are currently no operators for logical AND and OR .

## Functions

Functions that calculate a single field and filter functions can also be used in expressions, where they evaluate to the value of that field, or true or false for filter functions. However, function arguments are not automatically accepting expressions, even in the context of an expression.

For example, to calculate the cosine of a number we need to write n := 1 | c := math:cos(n) , because the cosine function expects a field name; the expression c := math:cos(1) computes the cosine of the value in the field named 1 .

Some CQL functions, like coalesce() , if() , getField() , and text:contains() , support expressions as arguments, but most functions take field or array names; the type of the parameter in the documentation will be "expression", if expressions are supported.

The function if is particularly useful in expressions, as it can be used to conditionally compute other expressions inside an expression.

## if() Function

The if() supports a typical if...then...else conditional statement. However, unlike a case or match statement, the if() can be embedded into other functions and expressions.

The if() supports statements like the one below where a comparison is being made between timestamps to determine the time of an error:

```
LOGSCALE 
errortime := if((@ingesttimestamp > @timestamp), then=@timestamp, else=@ingesttimestamp) / 1000
```

## Field Names in Expressions

In most places in a query, it is clear from the context whether a value or the name of a field is expected. In these cases, the name of a field can be written with or without quotation marks, meaning the same. For example, math:cos("fieldName") means the same as math:cos(fieldName) , because the argument to the function is defined as a string denoting a field name, and not an expression.

However, some field names cannot be written unquoted, because they are not bare words in the LogScale language; for example host-name or host/name need quotes when used in places where field names are expected: but do not: host.name=*, host[0]=* , and "host-name"=* all work, while host-name=* results in a syntax error.
```
@Note
The exact details of which names need quoting are complicated. The following table contains the allowed symbols marked with UnquotedFieldName: Character Table .
```

In expressions, on the other hand, quotation marks always mean a string value, while unquoted field names always mean the value of that field. To use the value of a field with such a name in an expression, the function getField() can be used with the quoted name, like coalesce([host, getField("host-name")]) . This works because getField() takes an expression and reads the value of the field with that name. Alternatively, if not using getField() , you can copy or rename the field before using it in an expression.

---
# Function Syntax

LogScale query functions process event data by producing, reducing, or modifying values within query pipelines, with support for both built-in functions and user-created functions through saved queries. The documentation covers function syntax including composite function calls, user functions with arguments, link functions for creating clickable results, and repeating queries that execute at regular intervals for dashboards and alerts.

LogScale query functions take a set of events, parameters, or configurations. From this, they produce, reduce, or modify values within that set, or in the events themselves within a query pipeline. LogScale has many built-in query functions. They are listed and described in the Query Functions section.

## Composite Function Calls

Whenever a function accepts a function as an argument, there are some special rules. For all variations of groupBy() , bucket() and timeChart() , that take a function= argument, you can also use a composite function. Composite functions take the form {f1(..) | f2(..) } which works like the composition of f1 and f2 . For example, you can do:


```
LOGSCALE SYNTAX 
groupBy(type, function={ avgFoo := avg(foo) | outFoo := round(avgFoo) })
```

You can also use filters inside such composite function calls, but not saved queries.

Incidentally, queries can contain Comments . This is useful for long multi-line queries and especially with saved queries, since later, you may not remember the purpose of each step. Below is an example of a comment embedded in a query:

```
LOGSCALE 
#type=accesslog // choose the type 
| top(url) // count urls and choose the most frequently used
```

## User Functions (Saved Searches)

In addition to built-in functions, you can create your own functions by saving a query that you use often. This can be used either as a saved query or saved search within other queries. Then it can be used as a top-level element of another query, similar to a Function Call .

To use a saved query this way you invoke it using the syntax $"SAVED\_QUERY\_NAME"() or, if the name does not contain whitespace or special characters you can use $nameOfSavedQuery() without quotes. A typical use for this is to define a filter or extraction ruleset that you can use as a prefix of another query.

Below is an example of how you might name and use a custom function:

```
LOGSCALE 
SYNTAX $"My Saved Query"() | $filterOutFalsePositive() 
| ...
```

To save a query within the UI, see Save queries .

## Using Arguments with User Functions

You can add arguments to your user functions by using the ?argname in your saved query; you then call the saved query and supply the argument name and matching value. For example, given the following query:

```
LOGSCALE 
host = ?host
```

Now save the query as findhost , you can execute the query using:

```
LOGSCALE 
$findhost(host="gendarme")
```

Multiple arguments can be added during the process. For example, when processing syslog data you can parse the content, create new fields, and then query that in the output:

```
LOGSCALE
regex(regex="Service exited due to (?<signal>\S+)")
| signal = ?signal
| regex(regex="sent by (?<process>\S+)[\d+]")
| process = ?process
```

Now we have two arguments which we can save to as a query killedprocess and then query for killed processes:

```
LOGSCALE 
$killedprocess(signal="SIGKILL", process="mds")
```

To call with only one argument, you can set a default value for the argument using ? {argument=default} , for example:

```
LOGSCALE 
?{signal="*" }
| ?{process="*"}
| regex(regex="Service exited due to (?<signal>\S+)")
| signal = ?signal
| regex(regex="sent by (?<process>\S+)\[\d+\]")
| process = ?process
```

Now you call the query with either argument:

```
LOGSCALE 
$killedprocess(process="mds")
```

## Using Functions as Arguments to Other Functions

Saved queries can be used in subqueries, passed to query functions that allow function arguments. However, saved queries used in such context must still meet the same requirements of the function they are in.

For example, saved queries can be used in functions such as stats() or groupBy() , like this:

```
LOGSCALE SYNTAX 
groupBy("myField", function=[count(), {id=42 
| $"My Saved Query"() }])
```

## Link Functions

When showing search results in a table it is possible to create a link that is clickable. If the value of a field looks like a link, the UI will make it clickable. Links can be constructed using the search language. The format() function can be handy for this.

```
LOGSCALE
$extractRepo() 
| top(repo) 
| format("https://example.com/%s", field=repo, as=link)
```

For further clarity, you can use Markdown formatting to give the link a title:

```
LOGSCALE 
$extractRepo() 
| top(repo) 
| format("[Link](https://example.com/%s)", field=repo, as=link)
```

## Repeating Queries

Some functions have limitations around their use in live queries, such as join() - see beta:repeating() and selfJoin() .

A repeating query is a static query that is executed at regular intervals and can be used in the same places as live queries, such as in dashboards or alerts.

You can turn a live query into a repeating by adding the beta:repeating() function to the query. For example, this query will be repeated every 10 minutes and can be used in dashboards and alerts:

```
LOGSCALE 
selfJoin(field=email_id, where=[{from=peter},{to=anders}]) 
| beta:repeating(10m)
```

Using beta:repeating() in a static query is allowed, but has no effect.

## Enabling Repeating Queries

Repeating queries are in beta. In order to use repeating queries, you must enable them by making the following GraphQL mutation as root from the API explorer found at $$YOUR\_LOGSCALE\_URL/docs/api-explorer :

```
GRAPHQL
mutation {
  enableFeature(feature: RepeatingQueries)
}
```

If this feature is disabled later, then any alert, dashboard, or saved query using beta:repeating() in the query will not function.




