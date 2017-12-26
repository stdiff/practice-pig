# Apache Pig 

- [Apache Pig](https://pig.apache.org/), 
  [Tutorial](https://www.tutorialspoint.com/apache_pig/) (tutorialspoint)
- There are four execution modes: local/mapreduce, with Tez/without Tez
  - MapReduce mode (default): `$ pig script.pig` 
  - [Tez mode](https://pig.apache.org/docs/r0.15.0/perf.html#tez-mode):
	`$ pig -x tez script.pig`
  - local mode: `$ pig -x local script.pig`
  - Tez local mode : `$ pig -x tez_local script.pig`
- The local mode runs without HDFS. 
- The Grunt shell starts if we give no script.
  - `grunt> sh ls` : same as `ls`
  - `grunt> fs -ls` : same as `hadoop fs -ls`

## Data Types

- int (32-bit), long (64-bit) / float (32-bit), double (64-bit) / chararray /
  Boolean (true,false) / Datetime (e.g. 1970-01-01T00:00:00.000+00:00)
- tuple : an ordered set of "fields" (an element). 
  *It represents a row of a relation (a table).* Ex. `(10, "a", 3.14)`
- bag : an unordered set of tuples. *It represents a relation (table).*
  Ex. `{(1,"a",1.4), (2,"b",1.7)}`
- The length of a tuple in a bag can vary. 
- map : something like a hash in Perl. `[key#value, key2#val2]`

## Projection operator

[Manual](https://pig.apache.org/docs/r0.15.0/basic.html#foreach). 
Let `b` be the following relation
	
	(setosa,     {(1,   5.1, 3.5, 1.4, 0.2, setosa),
	              (2,   4.9, 3.0, 1.4, 0.2, setosa), ... })
	(virginica,  {(150, 5.9, 3.0, 5.1, 1.8, virginica),
	              (101, 6.3, 3.3, 6.0, 2.5, virginica), ...})
	(versicolor, {(76,  6.6, 3.0, 4.4, 1.4, versicolor),
	              (52,  6.4, 3.2, 4.5, 1.5, versicolor),...})

And its schema is 
`group: chararray, a: {(id: int, x1: float, x2: float, x3: float, x4: float, species: chararray)}`.
Then

- `$0` and `group` are the first column consisting of chararray.
- `$1` and `a` are the second column consisting of bags.
- `$1.id` and `a.id` are the first elements of the bags.

## Data I/O

### LOAD

	a = LOAD 'basic_types.csv' USING PigStorage(';')
		    AS (id:int, time:datetime, score:int, prob:float, color:chararray);

- Schema on read. But we do not need to give it.
- **A schema is case-sensitive.**
- [Load/Store functions](https://pig.apache.org/docs/r0.15.0/func.html#load-store-functions)
- csv : `a = LOAD 'geolocation.csv' USING PigStorage(',');`
  - The quotation letter must be single quotation.
  - Without USING is equivalent to `USING PigStorage('\t')`
-  [CSVExcelStorage](http://pig.apache.org/docs/r0.13.0/api/org/apache/pig/piggybank/storage/CSVExcelStorage.html):
  enables an advanced option for loading a CSV file.
  - `REGISTER '/usr/hdp/2.6.0.3-8/pig/piggybank.jar';` is required before
    using it.
  - `USING org.apache.pig.piggybank.storage.CSVExcelStorage(';');` is the
    standard usage. This removes quotations.
  - `USING org.apache.pig.piggybank.storage.CSVExcelStorage(',', 'NO_MULTILINE', 'UNIX', 'SKIP_INPUT_HEADER');`:
	[skips the first line](https://stackoverflow.com/a/29335892/2205667) (the header).
- [Hive table](https://cwiki.apache.org/confluence/display/Hive/HCatalog+LoadStore):
  `a = LOAD 'geolocation' USING org.apache.hive.hcatalog.pig.HCatLoader();`
  - The option `-useHCatalog` is required.
  - The schema is automatically imported.
- [JSON Lines](http://jsonlines.org/)
  - `JsonLoader()` requires the order of the keys in the json string. In other
    words, keys are ignored.
  - [elephant-bird](https://github.com/twitter/elephant-bird) (to be tried)
- A general Pig relation : `BinStorage()`. This function requires a
  careful treatment of schema.


### STORE

	b = GROUP a BY id;
	c = FOREACH b GENERATE group as id, SUM(a.score) AS Total, AVG(a.score) AS Average;
	DUMP c;
	STORE c INTO 'store-test-dir' USING PigStorage (',');
	STORE c INTO 'pigtohive' USING org.apache.hive.hcatalog.pig.HCatStorer();
	
- The directory where CSV files are stored must not exist in advance.
- But **a Hive table must be created in advance and the schema must agree**.


## Diagnostic Operators

- [Manual](http://pig.apache.org/docs/r0.14.0/test.html)
- `DUMP c` : shows the relation
- `DESCRIBE c` : shows the schema : `c: {id: int,Total: long,Average: double}`.
- `EXPLAIN c` : shows an execution plan. 
- `ILLUSTRATE c` : shows a step-by-step execution of a sequence of statements.
  This is useful to see how the relation is changed in a script.


## Relational Algebra

[Manual](http://pig.apache.org/docs/r0.14.0/basic.html#Relational+Operators)

### For "rows" (FILTER, ORDER BY, etc)

	b = FILTER a BY prob > 0.5; -- pick the tuples satisfying the condition
	c = ORDER b BY score DESC;  -- arrange the tuples in descending order by score
	d = LIMIT c 3;              -- pick the top 3

- `col1 MATCHES '.*stryoulike.*'` is equivalent to `col1 LIKE '%stryoulike%'`
  of SQL. But as the name suggests, regex is applied. Note that we must gives
  a regular expression which matches the whole string.
- Note that `==` is used to compare two values unlike SQL.
- `is (not) null` is available
- The standard projection index (such as `$2`) can be used instead of the schema
  (such as `score`).
- `LIMIT c 4;` picks "first" 4 tuples in the relation `c`. You should use it
  in a combination with `DUMP` and/or `ORDER BY`.
- `DISTINCT c;` is equivalent to `SELECT DISTINCT * FROM c;` in SQL.
- `UNION c1, c2;` concatenates two relations, i.e. the union in the sense of
  relational algebra / SQL.
- `SPLIT a INTO b IF score >= 0, c IF score < 0;` creates two relations by
  given condisions. i.e. a combination of `FILTER` statements.
- `SAMPLE c 0.2;`

### For "columns" (FOREACH)

	b = FOREACH a GENERATE id, score, prob as rate, score * prob as prod;

`FOREACH ... GENERATE ...` corresponds to `SELECT` in SQL. 

The following function can be used to mutate data. 
([Manual](http://pig.apache.org/docs/r0.14.0/func.html)). 

- Maths:
  - well-known functions such as `ABS`, `SQRT`, `EXP`, `LOG`, `LOG10`, ...
  - the ceiling function `CEIL` and the floor function `FLOOR`.
  - `RANDOM()` produce a pseudo random number in $[0,1)$.
  - `ROUND(field)` rounds "field" to an integer.
  - `ROUND_TO(field,digits)` rounds "field" to a fixed number of decimal digits.
	E.g. `ROUND(3.15,1)` gives 3.2
- String:
  - `TRIM()`: removes leading/trailing white spaces.
  - `LOWER()`, `UPPER()` converts strings to lower/upper case.
  - `SUBSTRING(string,s,t)` is equivalent to `SUBSTRING(string,s,t-s)` in
    SQL. Namely "t" is the index of the end letter (not a length).
  - `EqualsIgnoreCase(str1,str2)` : compares two strings ignoring case
  - `CONCAT()` is the same as `CONCAT()` of MySQL.
  - `SPRINTF(format,arg1,arg2,...)` : formats strings. Use it if you want
  - `STRSPLIT(string,regex,limit)` splits a string with a regex and returns a
    tuple. Non-positive limit means no limits. But there is a difference
    between a negative limit and zero. 
		- `STRPLIT('accbc','c', 0)` returns `(a,b)`. 
 		- `STRPLIT('accbc','c',-1)` returns `(a,,b,)`. (Namely empty entries remain.)
  - `REGEX_EXTRACT(string,regex,index)` returns the substring which is matched
    the marked subexpression (i.e. regular expression with parentheses). If
    string is "a9aa8aa7a", then `REGEX_EXTRACT(string,'aa(\\d)a',1)` gives "8".
	Note that we must escape "\" in regexp, while we use no backslash for
    index, i.e. "1" instead of "\1" or "$1".
  - `REGEX_EXTRACT_ALL(string,regex)` returns the matched substrings as a
    tuple. Note that the regex must corresponds the whole string. That is 
	`REGEX_EXTRACT_ALL('a1b2c343', '.*?(\\d)(\\w).*')` returns `(1,b)`, while 
	`REGEX_EXTRACT_ALL('a1b2c343', '(\\d)(\\w)')` returns NULL.	
  - `REPLACE(string,regex,newstr)` replaces the matched part of the string
    with "newstr". Note that we can apply regular expression unlike
    `REPLACE()` of MySQL. (Thus `REGEXP_REPLACE()` of Oracle is equivalent to
    our `REPLACE()`.)
- Datetime:
  - `CurrentTime()` returns the current time.
  - `GetYear()`, `GetMonth()`, `GetDay()`, `GetHour()`, `GetMinute()`,
	`GetSecond()`
  - `GetWeek()` : returns the calender week
  -	`YearsBetween(t1,t0)`, `MonthsBetween(t1,t0)`, `WeeksBetween(t1,t0)`,
	`DaysBetween(t1,t0)`, `HoursBetween(t1,t0)`, `MinutesBetween(t1,t0)` : 
	they correspond to `TIMEDIFF(t1,t0)` in SQL. The prefix such as `Years`
	is a precision of the difference. (The returned values are always
	integer.)
  -	There is no buildin function for days of week, but the following statement
	works instead: `(DaysBetween(datetime,ToDate(0L)) + 4L) % 7`. This assigns
	0 to Sunday. The logic:
 	[1970-01-01 is Thursday](https://stackoverflow.com/a/22286579/2205667).
- NB: **A function name is case-sensitive.**


### Grouping 

	b = GROUP a BY color;                       -- {group: chararray, a: bag{...}}
	c = FOREACH b GENERATE group AS color, AVG(a.score) AS average; -- Aggregation 

 The schema of `b` is  `{group: chararray,a: {(id: int,time: datetime,score: int,prob: float,color: chararray)}}`.

- **The grouping field (column) becomes `group`.** (not "color" in the above case)
- The second element of a tuple is a bag consisting of the corresponding
  tuples in the original relation. The field of the bag is `a`, i.e. the name
  of the original relation.
- Therefore to reach a field in the bag, we need ".". In the above code the
  field "score" in the bag "a" can be reached through `a.score` or `a.$2`.

`GROUP ALL` is used to count the numbers of tuples.

	a_all = GROUP a ALL;                         -- consists of a single tuple: (all, a) 
	nrow = FOREACH a_all GENERATE COUNT_STAR(a); -- consists only of the number of tuples

We can use two fields for grouping.

	a1 = FOREACH a GENERATE color, (score>=0 ? 'pos' : 'neg') AS sign, prob;
	b = GROUP a1 BY (color, sign);
	c = FOREACH b GENERATE group.$0, group.$1, AVG(a1.prob) AS average;

The field `group` is a tuple consisting of the grouping fields. Note that the
schema of the tuple holds the names of the grouping fields. (Therefore we did
not write `group.$0 AS color` in the above code.)
  
Aggregate functions

- `AVG()`, `SUM()`, `MIN()` and `MAX()` ignores NULL values.
- `COUNT()` and `COUNT_STAR()` compute the number of tuples in a bag. The
  difference is `COUNT()` ignores NULL while `COUNT_STAR()` does not.


### COGROUP

Consider the following two relations `l` and `r`.

	-- l (id:int, dept:chararray)
	(110,HR)
	(120,Marketing)
	(130,Marketing)
	(140,Sales)
	
	-- r (dept:chararray,deptid:int)
	(Marketing,13)
	(Sales,17)
	(Development,20)

`COGROUP l by dept, r by dept;` applies `GROUP BY` for `l` and `r` and
combines them. 

	(HR,         {(110,HR)},                        {}                )
	(Sales,      {(140,Sales)},                     {(Sales,17)}      )
	(Marketing,  {(120,Marketing),(130,Marketing)}, {(Marketing,13)}  )
	(Development,{},                                {(Development,20)})

- Its schema is `{group: chararray, l: {schema of l}, r: {schema of r}}`. 
- If `a` is the above relation, then
  `FILTER a BY not IsEmpty(l) and not IsEmpty(r);`
  removes tuples with empty bag, so that the resulting relation is like an
  inner join.
  

### JOIN

`JOIN l by dept, r by dept;` returns the inner join of the two relations:

	(140, Sales,     Sales,     17)
	(120, Marketing, Marketing, 13)
	(130, Marketing, Marketing, 13)

- Its schema is `{l::id, l::dept, r::dept, r::deptid}`. 
- There is not `INNER JOIN` statement. Just `JOIN`.
- `a = COGROUP l by dept, r by dept; `

LEFT JOIN: `JOIN l by dept LEFT OUTER, r by dept;`

	(110, HR,        ,            )
	(140, Sales,     Sales,     17)
	(120, Marketing, Marketing, 13)
	(130, Marketing, Marketing, 13)

RIGHT JOIN: `JOIN l by dept RIGHT OUTER, r by dept;`

	(140, Sales,     Sales,       17)
	(120, Marketing, Marketing,   13)
	(130, Marketing, Marketing,   13)
	(,             , Development, 20)

OUTER JOIN: `JOIN l by dept FULL OUTER, r by dept;`

	(110, HR,        ,              )
	(140, Sales,     Sales,       17)
	(120, Marketing, Marketing,   13)
	(130, Marketing, Marketing,   13)
	(,             , Development, 20)

## Sample codes 

Count the number of rows by year:

	b = GROUP a BY year;
	c = FOREACH b GENERATE group AS year, COUNT_STAR(a.year) AS count;

## User defined functions (UDF)

- [Piggy Bank](https://pig.apache.org/docs/r0.15.0/udf.html#piggybank) is a
  place to share the Java UDFs.
- `/usr/hdp/2.6.0.3-8/pig/piggybank.jar`


## Performance 

- [Performance and Efficiency](https://pig.apache.org/docs/r0.15.0/perf.html), 
  [Pig Cookbook](https://pig.apache.org/docs/r0.8.1/cookbook.html)
- Specify the number of reduce tasks for a Pig MapReduce job
- `SET default_parallel 20;` : specifies the number of reducers.
- [Replicated join](https://pig.apache.org/docs/r0.15.0/perf.html#replicated-joins):
  if one of the relations to be joined are small enough, the small one is
  copied to all workers. 
  - Add `USING 'replicated'` at the end to perform a replicated join.
  - `C = JOIN big BY b1, tiny BY t1, mini BY m1 USING 'replicated';`
