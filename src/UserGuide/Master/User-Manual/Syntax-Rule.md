<!--

    Licensed to the Apache Software Foundation (ASF) under one
    or more contributor license agreements.  See the NOTICE file
    distributed with this work for additional information
    regarding copyright ownership.  The ASF licenses this file
    to you under the Apache License, Version 2.0 (the
    "License"); you may not use this file except in compliance
    with the License.  You may obtain a copy of the License at
    
        http://www.apache.org/licenses/LICENSE-2.0
    
    Unless required by applicable law or agreed to in writing,
    software distributed under the License is distributed on an
    "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
    KIND, either express or implied.  See the License for the
    specific language governing permissions and limitations
    under the License.

-->

# Syntax Rule

## Literal Values

This section describes how to write literal values in IoTDB. These include strings, numbers, timestamp values, boolean values, and NULL.

### String Literals

in IoTDB, **A string is a sequence of bytes or characters, enclosed within either single quote (`'`) or double quote (`"`) characters.** Examples：

```js
'a string'
"another string"
```

#### Usage Scenarios

Usages of string literals:

- Values of  `TEXT` type data in `INSERT` or `SELECT` statements 

    ```sql
    # insert
    insert into root.ln.wf02.wt02(timestamp,hardware) values(1, 'v1')
    insert into root.ln.wf02.wt02(timestamp,hardware) values(2, '\\')
    
    +-----------------------------+--------------------------+
    |                         Time|root.ln.wf02.wt02.hardware|
    +-----------------------------+--------------------------+
    |1970-01-01T08:00:00.001+08:00|                        v1|
    +-----------------------------+--------------------------+
    |1970-01-01T08:00:00.002+08:00|                        \\|
    +-----------------------------+--------------------------+
    
    # select
    select code from root.sg1.d1 where code in ('string1', 'string2');
    ```

- Used in`LOAD` / `REMOVE` / `SETTLE` instructions to represent file path.

    ```sql
    # load
    LOAD 'examplePath'
    
    # remove
    REMOVE 'examplePath'
    
    # SETTLE
    SETTLE 'examplePath'
    ```

- Password fields in user management statements

    ```sql
    # write_pwd is the password
    CREATE USER ln_write_user 'write_pwd'
    ```

- Full Java class names in UDF and trigger management statements 

    ```sql
    # Trigger example. Full java class names after 'AS' should be string literals.
    CREATE TRIGGER `alert-listener-sg1d1s1`
    AFTER INSERT
    ON root.sg1.d1.s1
    AS 'org.apache.iotdb.db.engine.trigger.example.AlertListener'
    WITH (
      'lo' = '0', 
      'hi' = '100.0'
    )
    
    # UDF example. Full java class names after 'AS' should be string literals.
    CREATE FUNCTION example AS 'org.apache.iotdb.udf.UDTFExample'
    ```

- `AS` function provided by IoTDB can assign an alias to time series selected in query. Alias can be constant(including string) or identifier.

    ```sql
    select s1 as 'temperature', s2 as 'speed' from root.ln.wf01.wt01;
    
    # Header of dataset
    +-----------------------------+-----------|-----+
    |                         Time|temperature|speed|
    +-----------------------------+-----------|-----+
    ```

- The key/value of an attribute can be String Literal and identifier, more details can be found at **key-value pair** part. 


#### How to use quotation marks in String Literals

There are several ways to include quote characters within a string:

 - `'` inside a string quoted with `"` needs no special treatment and need not be doubled or escaped. In the same way, `"` inside a string quoted with `'` needs no special treatment.
 - A `'` inside a string quoted with `'` may be written as `''`.
- A `"` inside a string quoted with `"` may be written as `""`.

The following examples demonstrate how quoting and escaping work:

```js
'string'  // string
'"string"'  // "string"
'""string""'  // ""string""
'''string'  // 'string

"string" // string
"'string'"  // 'string'
"''string''"  // ''string''
"""string"  // "string
```

### Numeric Literals

Number literals include integer (exact-value) literals and floating-point (approximate-value) literals.

Integers are represented as a sequence of digits. Numbers may be preceded by `-` or `+` to indicate a negative or positive value, respectively. Examples: `1`, `-1`.

Numbers with fractional part or represented in scientific notation with a mantissa and exponent are approximate-value numbers. Examples: `.1`, `3.14`, `-2.23`, `+1.70`, `1.2E3`, `1.2E-3`, `-1.2E3`, `-1.2E-3`.

The `INT32` and `INT64` data types are integer types and calculations are exact.

The `FLOAT` and `DOUBLE` data types are floating-point types and calculations are approximate.

An integer may be used in floating-point context; it is interpreted as the equivalent floating-point number.

### Timestamp Literals

The timestamp is the time point at which data is produced. It includes absolute timestamps and relative timestamps in IoTDB. For information about timestamp support in IoTDB, see [Data Type Doc](../Basic-Concept/Data-Type.md).

Specially, `NOW()` represents a constant timestamp that indicates the system time at which the statement began to execute.

### Boolean Literals

The constants `TRUE` and `FALSE` evaluate to 1 and 0, respectively. The constant names can be written in any lettercase.

### NULL Values

The `NULL` value means “no data.” `NULL` can be written in any lettercase.

## Identifier

### Usage scenarios

Certain objects within IoTDB, including `TRIGGER`, `FUNCTION`(UDF), `CONTINUOUS QUERY`, `SCHEMA TEMPLATE`, `USER`, `ROLE`,`Pipe`,`PipeSink`,`alias` and other object names are known as identifiers.

### Constraints

Below are basic constraints of identifiers, specific identifiers may have other constraints, for example, `user` should consists of more than 4 characters. 

- Permitted characters in unquoted identifiers:
    - [0-9 a-z A-Z _ ] (letters, digits and underscore)
    - ['\u2E80'..'\u9FFF'] (UNICODE Chinese characters)
- Identifiers may begin with a digit, unquoted identifiers can not be a real number.
- Identifiers are case sensitive.
- Key words can be used as an identifier.

**You need to quote the identifier with back quote(`) in the following cases:**

- Identifier contains special characters.
- Identifier that is a real number.

### How to use quotations marks in quoted identifiers

`'` and `"` can be used directly in quoted identifiers.

` may be written as `` in quoted  identifiers. See the example below:

```sql
# create template t1't"t
create schema template `t1't"t` 
(temperature FLOAT encoding=RLE, status BOOLEAN encoding=PLAIN compression=SNAPPY)

# create template t1`t
create schema template `t1``t` 
(temperature FLOAT encoding=RLE, status BOOLEAN encoding=PLAIN compression=SNAPPY)
```

### Examples

Examples of case in which quoted identifier is used ：

- Trigger name should be quoted in cases described above ：

    ```sql
    # create trigger named alert.`listener-sg1d1s1
    CREATE TRIGGER `alert.``listener-sg1d1s1`
    AFTER INSERT
    ON root.sg1.d1.s1
    AS 'org.apache.iotdb.db.storageengine.trigger.example.AlertListener'
    WITH (
      'lo' = '0', 
      'hi' = '100.0'
    )
    ```

- UDF name should be quoted in cases described above ：

    ```sql
    # create a funciton named 111, 111 is a real number.
    CREATE FUNCTION `111` AS 'org.apache.iotdb.udf.UDTFExample'
    ```

- Template name should be quoted in cases described above ：

    ```sql
    # create a template named 111, 111 is a real number.
    create schema template `111` 
    (temperature FLOAT encoding=RLE, status BOOLEAN encoding=PLAIN compression=SNAPPY)
    ```

- User and Role name should be quoted in cases described above, blank space is not allow in User and Role name whether quoted or not ：

    ```sql
    # create user special`user.
    CREATE USER `special``user.` 'write_pwd'
    
    # create role 111
    CREATE ROLE `111`
    ```

- Continuous query name should be quoted in cases described above ：

    ```sql
    # create continuous query test.cq
    CREATE CONTINUOUS QUERY `test.cq` 
    BEGIN 
      SELECT max_value(temperature) 
      INTO temperature_max 
      FROM root.ln.*.* 
      GROUP BY time(10s) 
    END
    ```

- Pipe、PipeSink should be quoted in cases described above ：

    ```sql
    # create PipeSink test.*1
    CREATE PIPESINK `test.*1` AS IoTDB ('ip' = '输入你的IP')
    
    # create Pipe test.*2
    CREATE PIPE `test.*2` TO `test.*1` FROM 
    (select ** from root WHERE time>=yyyy-mm-dd HH:MM:SS) WITH 'SyncDelOp' = 'true'
    ```

- `AS` function provided by IoTDB can assign an alias to time series selected in query. Alias can be constant(including string) or identifier.

    ```sql
    select s1 as temperature, s2 as speed from root.ln.wf01.wt01;
    
    # Header of result dataset
    +-----------------------------+-----------|-----+
    |                         Time|temperature|speed|
    +-----------------------------+-----------|-----+
    ```

- The key/value of an attribute can be String Literal and identifier, more details can be found at **key-value pair** part. 
  
- Time series path supports the creation of hierarchical paths containing "*" for paths other than the database level, e.g. root.db.`*`.


## KeyWords Words

Keywords are words that have significance in SQL. Keywords can be used as an identifier. Certain keywords, such as TIME/TIMESTAMP and ROOT, are reserved and cannot use as identifiers.

[Keywords](../Reference/Keywords.md) shows the keywords in IoTDB.

## Detailed Definitions of Lexical and Grammar

Please read the lexical and grammar description files in our code repository:

Lexical file: `antlr/src/main/antlr4/org/apache/iotdb/db/qp/sql/IoTDBSqlLexer.g4`

Grammer file: `antlr/src/main/antlr4/org/apache/iotdb/db/qp/sql/IoTDBSqlParser.g4`
