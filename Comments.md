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

## Operators

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

## Adding Fields

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


