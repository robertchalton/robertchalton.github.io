---
layout: post
published: true
title: 'SQLcl Format options in 19.1'
date: '2019-04-18'

---


# SQLcl 19.1 sqlformat options

The biggest change for was that the help was overhauled to include description and examples.  While some format like `csv` are very straight forward on what to expect others like `loader` or `delimited` are not. Hopefully this make things like `delimited` a lot more clear on how to set the left , right , and separator characters.

The only new format is now there's a `json-formatted` which is meant to be more readable format by doing a pretty print to the screen.

```
SQL> help set sqlformat
SET SQLFORMAT
  SET SQLFORMAT { default,csv,html,xml,json,fixed,insert,loader,delimited,ansiconsole}   
   
   default        : SQL*PLUS style formatting 
   csv            : comma separated and string enclosed with " 
   html           : html tabular format 
   xml            : xml format of /results/rows/column/* 
   json           : json format matching ORDS Collection Format 
   json-formatted : json format matching ORDS Collection Format and pretty printed 
   fixed          : fixed width 
   insert         : generates insert statements from sql results 
                    Example 
                      Insert into EMP (EMPNO,ENAME,JOB,MGR,HIREDATE,SAL,COMM,DEPTNO) 
                      values (7369,'SMITH','CLERK',7902,to_timestamp('17-DEC-80','DD-MON-RR HH.MI.SSXFF AM'),800,null,20); 

   loader         : pipe (|) delimited enclosed with "  
                    Example:  
                       7369|"SMITH"|"CLERK"|7902|"1980-12-17 00:00:00"|800||20|5555555555554444| 

   delimited      : CSV format with optional separator , left, and right enclosure  
                    set sqlformat delimited [separator] [left enclosure] [right enclosure] 
                    Example:  
                    set sqlformat delimited , < >    
                       7369,<SMITH>,<CLERK>,7902,17-DEC-80,800,,20,5555555555554444 

   ansiconsole    : advanced formatting based on data and terminal size 
                    set sqlformat ansiconsole                       : base format 
                    set sqlformat ansiconsole default               :  number formatting to ###,###.### 
                    set sqlformat ansiconsole <number format mask>  : Mask following Java DecimalFormat 

                               https://docs.oracle.com/javase/8/docs/api/java/text/DecimalFormat.html  

                    set sqlformat ansiconsole -config=highlight.json : highlight matches in results 

                    highlight options : 
                    Example :  
                    {"highlights":[ 
                        {"type":"startWith","test":"W","color":"INTENSITY_BOLD,CYAN"},
                        {"type":"endsWith","test":"MAN","color":"BLUE"},
                        {"type":"contains","test":"MIT","color":"YELLOW"},
                        {"type":"exact","test":"FORD","color":"GREEN"},
                        {"type":"regex","test":"[0-9]{2}","color":"MAGENTA"}
                      ]
                    } 

   
SQL> 
```
# json-formatted

The new `json-formatted` is purty.

```


SQL> set sqlformat json-formatted
SQL> select * from emp;
{
  "results" : [
    {
      "columns" : [
        {
          "name" : "EMPNO",
          "type" : "NUMBER"
        },
        {
          "name" : "ENAME",
          "type" : "VARCHAR2"
        },
        {
....
```                    

# ansiconsole

I spent some time getting the headers and spacing more aligned so the ansiconsole format will be aligned as expected there were a number of cases where this wasn't happening.

The larger change here is to add the highlighting feature.  The help has a full example of the options that are useful as a test against a `select * from emp` table.  Yes theres is a bug here that is the example has `endWith` and should be `endsWith`

![](/img/sqlcl_ansi_highlight.png)

The types in this are a direct usage of the underlying [java.lang.String](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/String.html) methods with in some cases the same names 

exact      - This is not a function rather a simple equality check and is case 
sensative.

[startsWith](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/String.html#startsWith(java.lang.String)) - Tests if this string starts with the specified prefix.

[endsWith](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/String.html#endsWith(java.lang.String))   - Tests if this string ends with the specified suffix.

[contains](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/String.html#contains(java.lang.CharSequence))   - Returns true if and only if this string contains the specified sequence of char values.


[regex](https://docs.oracle.com/en/java/javase/12/docs/api/java.base/java/lang/String.html#matches(java.lang.String))      - Tells whether or not this string matches the given regular expression.

One thing to note is the config file is re-read with every execution to make iterating the setting easier vs caching it at the time of the sqlformat command being issued.

