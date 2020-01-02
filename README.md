# Xlsx2Json

[![Build Status](http://jenkins.chewrobot.com/job/Xlsx2Json/badge/icon)](http://jenkins.chewrobot.com/job/Xlsx2Json/)

A Java parser to convert xlsx sheets to JSON

Supported platforms: Anywhere you can run a Java program

## Quick start

1. Download release or use Gradle to make a build
2. Run the command line (See [Usage](#usage) section)
3. The json file will be generated with the same filename

## Usage

> java -jar xlsx2json-x.x.jar target_name "sheet_name_1 sheet_name_2 ..." [true|false]

Example

> java -jar xlsx2json-1.2.jar test.xlsx "monsters maps weapons" true

### Arguments

* The first argument is the Excel filename
* The second argument is the sheet you want to export to json
* The third argument indicates whether show the sheet names in generated json or not, will be set to false if omitted

e.g.
> ture = {"sheet1":{...},"sheet2":{...}
> false = [{...},{...}]

## Gradle build command

> $ gradle clean

> $ gradle fatJar

The Jar is created under the ```$project/build/libs/``` folder.

## Input file format

The first row should be column type definition (See [Supported types](#supported-types) section)

The second row should be column name definition

The following rows should be data

> **Especially**, the first column should be **Basic** type so the parser could index it as the primary key

## Example (Excel .xlsx file)

### Monsters sheet

| Integer | String | Basic  | Array\<Double\> | Array\<String\>   | Reference   | Object      |
| ----   | --------| ------ | ---------------- | ---------- | ---------- | ------------ |
| id     | weapon  | $flag   | nums  | words  | shiled@shield#$id   | objects      |
| 123    | shield  | TRUE   | 1,2   | hello,world   | 123 | a:123,b:"45",c:false   |
|        | sword   | FALSE  | null  | oh god       |   | a:11;b:"22",c:true    |
![monsters sheet](https://raw.githubusercontent.com/noahzark/Xlsx2Json/master/example/monsters.png)

### Shields sheet

> The type definition line is omitted because all columns are basic types

| $id     | name  | forSale   | price  |
| ----   | --------| ------ | ------ | 
| 123    | COPPER SHIELD  | TRUE   | 3600 |

![shields sheet](https://raw.githubusercontent.com/noahzark/Xlsx2Json/master/example/shields.png)

Result:

```json
{
   "monsters":[
      {
         "weapon":"shiled",
         "objects":{
            "a":123,
            "b":"45",
            "c":false
         },
         "words":[
            "hello",
            "world"
         ],
         "id":123,
         "nums":[
            1,
            2
         ],
         "shiled":{
            "forSale":true,
            "price":3600,
            "name":"COPPER SHIELD",
         }
      },
      {
         "weapon":"sword",
         "objects":{
            "a":11,
            "b":"22",
            "c":true
         },
         "words":[
            "oh god"
         ],
         "id":null,
         "nums":[

         ],
         "shiled":null
      }
   ]
}
```

## Supported types

### Basic Types

* String
* Integer
* Float
* Double
* Boolean

You can use "Basic" to let the parser automatically detect types

> **Especially** if all columns are **Basic** types, you can omit the type definition row

### Time Type

Support all strings with format **HH:mm:ss** (directly out) or cell format **Time** (converted to a calander object and format with simple date format)

### Date Type
Support string or numeric with format **yyyyMMdd** and it will format as **yyyy-MM-dd** in the json.

> **DateTIme** support would be added in future versions, add an issue if you need it.

### Array Types

* Array\<String\>
* Array\<Boolean\>
* Array\<Double\>

The values should be divided using commas ","

> You can use the Array\<Double\> to represent all numeric types like Integer/Float and so on

### Object type

* Object

Use this one to directly construct a JSON object using basic types, child should be divided using commas ","

> For more complicated objects, see Reference type

### Reference type

* Reference

Use this type to insert a JSON object from another sheet, the format should be

```name_of_this_column@sheet_name#column_name```

and the value should be the column value of target.

Use **@** to split column name and sheet name, use **#** to split target sheet name and target column name

### Null values

* Null

If a column is blank, will automatically generate a null value in the JSON file.

### Hidden columns

* $column_name

If a column's name starts with **$** sign, then it won't appear in the result json

> **Especially**, if you want to **reference** to a hidden column, you should also include the **$** sign in reference column name
