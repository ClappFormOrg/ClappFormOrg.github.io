

# Query

The query editor is meant to create a pipeline that transforms data at the latest possible stage before presentation. This means it is intended for selecting only the data necessary for creating a widget.

# Pipeline

The top layer of the query editor consists of the stages together forming the pipeline. These stages can be used in any quantity and order, but caution is advised for the sake of time-to-live of the widget. At the start of the pipeline the data will be passed down of the collection selected in the form. The data for the last stage will be passed down to the widget and can then be configured in the widget configuration. The last stage will always be limited to 100 items maximum to guarantee performance. The stages can have multiple different types:

*	[`add field`](#add-field)
*	[`array to object`](#array-to-object)
*	[`bucket`](#bucket)
*	[`bucket auto`](#bucket-auto)
*	[`calculation`](#calculation)
*	[`convert`](#convert)
*	[`filters`](#filters)
*	[`first from array`](#first-from-array)
*	[`group`](#group)
*	[`hide`](#hide)
*	[`kpis`](#kpis)
*	[`left join`](#left-join)
*	[`limit`](#limit)
*	[`object to array`](#object-to-array)
*	[`order`](#order)
*	[`raw`](#raw)
*	[`rename`](#rename)
*	[`show`](#show)
*	[`to Date`](#to-date)
*	[`to Epoch`](#to-epoch)
*	[`union with`](#union-with)
*	[`unwind`](#unwind)



A full example could be
```json
[
  {
    "filters": [
      {
        "regio": [
          {
            "type": "eq",
            "value": "Nederland"
          }
        ]
      }
    ],
    "type": "filters"
  },
  {
    "steps": [
      {
        "field": "periode",
        "how": "DESC"
      }
    ],
    "type": "order"
  },
  {
    "amount": 1,
    "type": "limit"
  }
]
```

# Values

If a value needs to be added to the query one may use the following types: string (text), number or value from a key. Text needs to be surrounded by double quotes and to select the value from a key the name needs to be proceeded by a $ sign. For example `"value": "$price"`.

# Global Values

The following system variables are provided for use in expressions.

Value| Description
---|---
`$USER` | ID of the currently active user. Used to make the dashboard user specific
`$DEEP_DIVE` | Provides access to the values of a chart when clicking on a specific chart. For example, if the chart contains information regarding an address and you want to use this key value use: `"$DEEP_DIVE.address"`
`$ID` | Refers to the identifier passed in the URL. Used in details pages to specify what item is currently being viewed. Both input and output are controlled by the settings of the sending widget and receiving page
`$$NOW` | Returns current datatime value, this remains constant throughout the aggregation pipeline
`$$REMOVE`  | Allows for conditional exclusion of fields

# Stage types

## [**Add field**](#pipeline)

Adds new fields to object.


Name | Value
---|---
fields	| Object(Where the key is the new name and the value is value you want to add )

#### Example:
```json
{
  "type": "addField",
  "fields": {
    "new": "value"
  }
}
```

The value field also accepts the following as input:

Value | Descriptions
---|---
`$GET_DATE_NOW`	| Returns the current date as `"28-09-2022"` (String)
`$GET_DATE_SEC`	| Returns the current time as seconds (Unsigned Int)
`$GET_DATE_MIL`	| Returns the current time as milliseconds (Unsigned Long Long)
`$GET_DATE_MIC`	| Returns the current time as microseconds (Unsigned Long Long)
`$GET_DATE_NAN` | Returns the current time as nanoseconds (Unsigned Long Long)


## [**Array to object**](#pipeline)

Converts an array into a single document;


Name | Value
---|---
fields	| Object(Where the key is the new name and the value is the old name)

#### Example:
```json
{
  "type": "arrayToObject",
  "fields": {
    "new": "old"
  }
}
```

## [**Bucket**](#pipeline)

Categorizes incoming documents into groups, called buckets, based on a specified expression and bucket boundaries and outputs a document per each bucket. Each output document contains an `_id` field whose value specifies the inclusive lower bound of the bucket. The output option specifies the fields included in each output document.

Bucket only produces output documents for buckets that contain at least one input document.

Name | Type | Description
--- | --- | ---
groupBy	| Text | An expression to group documents by.
boundaries	| Array | An array of values based on the groupBy expression that specify the boundaries for each bucket.
default | Text | Optional. A literal that specifies the `_id` of an additional bucket that contains all documents whose groupBy expression result does not fall into a bucket specified by boundaries.
output | Object | Optional. A document that specifies the fields to include in the output documents in addition to the `_id` field.

For example, a boundries array of `[ 0, 5, 10 ]` creates two buckets:
- `[0, 5]` with inclusive lower bound `0` and exclusive upper bound `5`.
- `[5, 10]` with inclusive lower bound `5` and exclusive upper bound `10`.

### Example:
```json
{
  "groupBy" : "id",
  "boundaries" : [ 0, 5, 10 ],
  "default" : "remaining",
  "output": {
    "count": { "$sum": 1 },
    "avg": { "$avg": "$id" }
  },
  "type" : "bucket"
}
```
```json
{
  "groupBy" : "id",
  "boundaries" : [ 0, 5, 10 ],
  "default" : "remaining",
  "output" : {
    "count": { "$sum": 1 },
    "avg": { "$avg": "$id" },
    "routers": { "$addToSet": "$router"  },
    "data": {
      "$push": {
        "method": {
          "$concat": [ "$method", " ", "$path" ]
        },
        "status": "$status"
      }
    }
  },
  "type" : "bucket"
}
```

## [**Bucket auto**](#pipeline)

Categorizes incoming documents into a specific number of groups, called buckets, based on a specified expression. Bucket boundaries are automatically determined in an attempt to evenly distribute the documents into the specified number of buckets.

Name | Type | Description
--- | --- | ---
groupBy	| Text | An expression to group documents by.
buckets	| Number | A positive 32-bit integer that specifies the number of buckets into which input documents are grouped.
output	| Object | Optional. A document that specifies the fields to include in the output documents in addition to the `_id` field.
granularity	| Text | Optional. A string that specifies the [preferred number series](https://en.wikipedia.org/wiki/Preferred_number) to use to ensure that the calculated boundary edges end on preferred round numbers or their powers of 10.

The suppported values of `granularity` are:
* `"R5"`
* `"R10"`
* `"R20"`
* `"R40"`
* `"R80"`
* `"1-2-5"`
* `"E6"`
* `"E12"`
* `"E24"`
* `"E48"`
* `"E96"`
* `"E192"`
* `"POWERSOF2"`

### Example:
```json
{
  "groupBy": "id",
  "buckets": 6,
  "output": {
    "count": { "$sum": 1 },
    "avg": { "$avg": "$id" }
  },
  "granularity": "R80",
  "type" : "bucketAuto"
}
```

## [**Calculation**](#pipeline)

This stage has been created in order to be able to create new columns derived from existing columns. Usage could be appending text to text, doing basic calculations or creating categories for numerical records.

The different types of calculations are:
* `add`
* `subtract`
* `divide`
* `multiply`
* `abs`
* `avg`
* `ceil`
* `concat`
* `floor`
* `log` (Usage: [num, base])
* `ltrim`
* `rtrim`
* `trim`
* `max`
* `mod` (modulo)
* `pow` (Usage: [num, power])
* `round` (Usage: [num, decimals])
* `sum`
* `sd` (Standard Deviation)
* `switch`

### Examples
Single calculation `(basic_calculation = $price * 2)`
```json
{
  "type": "calculation",
  "fields": {
    "basic_calculation": {
      "multiply": ["$price", 2]
    }
  }
}
```

Multilayered `(multi_calculation = (price + abs(-10)) / 4)`
```json
{
  "type": "calculation",
  "fields": {
    "multi_calculation": {
      "divide": [{
        "add": ["$price", {
          "abs": -10
        }]
      }, 4]
    }
  }
}
```

Switch statement
```json
{
  "type": "calculation",
  "fields": {
    "segment": {
      "switch": [
        {
          "if": [{
            "gte": ["$price", 1500]
          }],
          "then": "Upper"
        },
        {
          "default": true,
          "then": "Lower"
        }
      ]
    }
  }
}
```

## [**Convert**](#pipeline)

Converts a value to a specified type.

Name | Value
--- | ---
input | Text
to |	Text
onError |	Text (Optional)
onNull |	Text (Optional)


The 'to' argument can be any valid expression that resolves to one of the following string identifiers:
*	`double`
*	`string`
*	`objectId`
*	`bool`
*	`date`
*	`int`
*	`long`
*	`decimal`
*	`euro`

#### Example:

```json
{
  "type": "convert",
  "input": "timestamp",
  "to": "string",
  "onError": "na",
  "onNull": "null"
}
```

## [**Filters**](#pipeline)

The filters stage's intended purpose is to query the data and return all matching records. One can i.e. select only country specific data. There is support for AND statements and OR statements on multiple fields. Every expression needs to contain a `type` and `value`.

Different types are:
* `eq` (Equal to)
* `ne` (Not equal to)
* `lt` (Less than)
* `lte` (Less than or equal to)
* `gt` (Greater than)
* `gte` (Greater than or equal to)

### Examples:
Where region equals Nederland and price is greater than 1000
```json
{
  "filters": [
    {
      "regio": [
        {
          "type": "eq",
          "value": "Nederland"
        }
      ],
      "price": [
        {
          "type": "gt",
          "value": 1000
        }
      ]
    }
  ],
  "type": "filters"
}
```

Where region equals Nederland or Duitsland and price is greater than 1000
```json
{
  "filters": [
    {
      "regio": [
        {
          "type": "eq",
          "value": "Nederland"
        }
      ],
      "price": [
        {
          "type": "gt",
          "value": 1000
        }
      ]
    },
    {
      "regio": [
        {
          "type": "eq",
          "value": "Duitsland"
        }
      ],
      "price": [
        {
          "type": "gt",
          "value": 1000
        }
      ]
     }
  ],
  "type": "filters"
}
```
Where region equals Nederland and price is greater than 1000 and less than the market_value
```json
{
  "filters": [
    {
      "regio": [
        {
          "type": "eq",
          "value": "Nederland"
        }
      ],
      "price": [
        {
          "type": "gt",
          "value": 1000
        },
        {
          "type": "lt",
          "value": "$market_value"
        }
      ]
    }
  ],
  "type": "filters"
}
```
## [**First from array**](#pipeline)
Get first element from an array. Checks if the selected column is an array, if not then returns the complete array

Name | Value
---|---
fields	| Object (The key is the new name for the array item and the value is the complete array column)

#### Example:

```json
{
  "type": "firstFromArray",
  "fields": {
    "newKey": "sourceArray"
  }
}
```

## [**Group**](#pipeline)

In order to create calculated values over multiple items the group stage exists. In this stage the group keys must be declared together with the calculated keys it must retain. All keys not set here will not be passed down. Furthermore, the Group stage does not sort its results, the Order stage can be used to sort the results.

Name | Value
---|---
by | List (Keys that together need to form a new unique item)
fields | Object

The field needs to get a name, type of calutation and the column on which to perform the calculation.
The possible options for the type of fields include:
 * `sum`
 * `avg` (Average/ median)
 * `min` (Minimum)
 * `max` (Maximum)
 * `sdp` (Standard Deviation Population)
 * `sds` (Standard Deviation Sample)
 * `first` (First value)
 * `last` (Last value)
 * `list`
 * `ulist` (Unique List)
 * `count` (Total values)
 * `ucount` (Total of unique values)

#### Example:

```json
{
  "type": "group",
  "by": ["postcode"],
  "fields": {
      "total_unique": {
          "type": "ucount",
          "column": "house_price"
      },
      "address": {
          "type": "first",
          "column": "address"
      }
  }
}
```
## [**Hide**](#pipeline)
Removes/excludes fields from documents. (For the inverse, see [`show`](#show)) 

Name | Value
--- | ---
fields | Array (List containing names of the keys to remove)

#### Example:

```json
{
  "type": "hide",
  "fields": ["listing_photo_urls", "references", "postcode"]
}
```

## [**Kpis**](#pipeline)

In order to create calculated values over all the remaining records the kpis stage exists. In this stage the kpi keys must be declared. All keys not set here will not be passed down.

Name | Value
---|---
fields | Object

The field needs to get a name, type of calutation and the column on which to perform the calculation.
The possible options for the type of fields include:
 * `sum`
 * `avg` (Average/ median)
 * `min` (Minimum)
 * `max` (Maximum)
 * `sdp` (Standard Deviation Population)
 * `sds` (Standard Deviation Sample)
 * `first` (First value)
 * `last` (Last value)
 * `list`
 * `ulist` (Unique List)
 * `count` (Total values)
 * `ucount` (Total of unique values)

#### Example:
```json
{
  "type": "kpis",
  "fields": {
      "total": {
          "type": "count",
          "column": "id"
      }
  }
}
```

## [**Left join**](#pipeline)

This type is intended for joining data from a collection within the same app. This would be used if the data necessary for a table is split up in 2 or more collections (i.e. employee and manager). Below are the possible options to pass to this stage.

Name | Value
--- | ---
collection | Text
how |	Text (inner or outer, defaults to inner)
left_key |	Text (key in the previous stage)
right_key |	Text (key of the selected collection)

#### Example:

```json
{
  "type": "left join",
  "how": "outer",
  "collection": "1017",
  "left_key": "adress",
  "right_key": "adress"
}
```

The "left join" step also accepts "$USER_TABLE" as a collection value. When this identifier is used the system replaces the specified left_key. The right_key must then specify whether the left_key is an ID (integer) or string. For example, the following dataset:
```json
{
  "location": "work",
  ...
  "user_id": 1,
  "user_email": "j.doe@clappform.com"
}
```
With this query:
```json
{
  "type": "left join",
  "collection": "$USER_TABLE",
  "left_key": "user_id", // alt: "left_key": "user_email"
  "right_key": "id" // alt: "right_key": "email"
}
```

Returns:
```json
{
  "location": "work",
  ...
  "user_id": {
    "name": "john",
    "lastname": "doe",
    "email": "j.doe@clappform.com"
  },
  "user_email": "j.doe@clappform.com"
}
```

## [**Limit**](#pipeline)

This type will limit the amount of items passed down to the next stage.

Name | Value
---|---
amount |	Number


## [**Object to array**](#pipeline)

Converts a object to an array. The return array contains an element for each field/value pair in the original document.


Name | Value
---|---
fields	| Object(Where the key is the new name and the value is the old name)

#### Example:
```json
{
  "type": "objectToArray",
  "fields": {
    "new": "old"
  }
}
```

## [**Order**](#pipeline)

The order stage allows you to order all passed items on multiple fields. The fields object stores two arrays that contain the sorting methods. The order stage is limited to 30 keys. Each array contains a list of keys that the query will sort on. For example, if the query only needs to sort ascending by ID, then the stage will look like this:

```json
{
  "type": "order",
  "fields": {
    "ASC": ["id"]
  }
}
```

Name | Value
---|---
fields	| Text (Object containing two arrays)

#### Example:

```json
{
  "type": "order",
  "fields": {
      "ASC": ["id", "province"],
      "DESC": ["municipality"]
  }
}
```

## [**Raw**](#pipeline)
The raw stage provides a way to add more complex stages to the query if the currently available stages are not sufficient. The queries are performed on an [MongoDB](https://www.mongodb.com/docs/v6.0/) running version 6.0.1, all queries are added to the aggregation stage used to retrieve the data. We advise caution when using this stage as the queries proper knowledge on the workings of MongoDB queries.

Name | Value
---|---
stages | Array (Contains all manually added stages)


```json
{
   "type":"raw",
   "stages":[
      {
         "$addFields":{
            "ref_1.publication_date":{
               "$cond":{
                  "else":"$$REMOVE",
                  "if":"ref_1: { $exists: true }",
                  "then":{
                     "$toDate":"$ref_1.publication_date"
                  }
               }
            }
         }
      },
      {
        "$unwind": {
          "path": "$buckets",
          "preserveNullAndEmptyArrays": true
        }
      }
   ]
}
```

## [**Rename**](#pipeline)

This type is used for renaming the keys to pass to the next stage.

Name | Value
--- | ---
fields	| Object (The key is the new name and the value is the old name)
check_for_arrays | Boolean (Optional. If renamed object is a array the elements will be concated to a single string). Default: True

#### Example:

```json
{
  "type": "rename",
  "fields": {
    "new": "old"
  }
}
```


## [**Show**](#pipeline)

This type is used for selecting keys to pass to the next stage. (For the inverse, see [`hide`](#hide))

Name | Value
--- | ---
columns	| Array (List containing names of the keys to keep)

#### Example:

```json
{
  "type": "show",
  "fields": ["new_acceptance", "adress", "building_year"]
}
```


## [**To Date**](#pipeline)

Inverse of toEpoch. Converts an integer timestamp to a data string. Additionally accepts format specifiers to modify the output date string, if 'format' is not specified the stage default is `"%Y-%m-%d"` which outputs as `"2022-06-15"`. You can also specify the timezone, if 'timezone' is not specified the stage default is `"Europe/Amsterdam"`.

Name | Value
---|---
column	| Text
format	| Text (Optional)
timezone	| Text (Optional)

### Format Specifiers

Specifiers | Description | Possible Values
---|---|---
%d	| Day of Month (2 digits, zero padded) | 01-31
%G	| Year in ISO 8601 format | 0000-9999
%H	| Hour (2 digits, zero padded, 24-hour clock) | 00-23
%L	| Millisecond (3 digits, zero padded) | 000-999
%m	| Month (2 digits, zero padded) | 01-12
%M	| Minute (2 digits, zero padded) | 01-31
%S	| Second (2 digits, zero padded) | 00-60
%u	| Day of week number in ISO 8601 format (1-Monday, 7-Sunday) | 1-7
%v	| Week of Year in ISO 8601 format | 1-53
%Y	| Year (4 digits, zero padded) | 0000-9999
%z	| The timezone offset from UTC. | +/-[hh][mm]
%Z	| The minutes offset from UTC as a number. For example, if the timezone offset (+/-[hhmm]) was +0445, the minutes offset is +285. | +/-mmm
%%	| Percent Character as a Literal | %

#### Example:
Format shown below ouputs as `2022-06-15 11:06:52`
```json
{
  "type": "toDate",
  "column": "datestring",
  "format": "%Y-%m-%d %H:%M:%S", 
  "timezone": "Europe/Amsterdam"
}
```

## [**To Epoch**](#pipeline)

Converts key to epoch timestamp. This stage expects a datestring as input and returns epoch timestamp.

Name | Value
---|---
column	| Text

#### Example:
```json
{
  "type": "toEpoch",
  "column": "datestring"
}
```

## [**Union With**](#pipeline)

Performs a union of two collections; i.e. 'union with' combines results from two collections into a single result set.


Name | Value
---|---
collection	| Text (Collection slug)
pipeline| Array (Optional. An aggregation pipeline to apply to the specified collection). All references to keys should include 'data.' as the data is not saved within the root, i.e. if the column is named 'firstname' the name is 'data.firstname'

#### Example:
```json
{
  "type": "unionWith",
  "collection": "feedback_resultaten",
  "pipeline": [ { "$project": { "beschrijving": 1 } } ]

}
```
This could result in duplicates, to remove duplicates you can use the 'group' stage type.

#### Example:
```json

{
  "type": "unionWith",
  "collection": "collection1"
},
{
"type": "group",
"by": ["email"],
"fields": {
    "email": {
        "type": "first",
        "column": "email"
    }
  }
}

```

## [**Unwind**](#pipeline)

Deconstructs an array field from the input documents to output a document for each element. Each output document is the input document with the value of the array field replaced by the element.


Name | Value
---|---
fields	| Array (List containing names of the keys)

#### Example:
```json
{
  "type": "unwind",
  "fields": ["feedback_resultaten"]
}
```

# Questionnaire
To configure the relevant questionnaire for the user, you can write a query in order determine which data will be loaded in the questionnaire. If you don't configure a query please be aware it will always parse in data if the collection has data. Making a questionnaire user specific, or other condition use the query options in order to fulfill your goal.

# Explanation
In several places of the widget configuration you have the possibility to provide your own text using a WYSIWYG. It is possible to parse in data from the query into your text. In every case only the first item of the query result is accessible. Let's say you want to parse in the address in your text, you need to do the following:

## Example:
`{% highlight html%}`
`{% raw %}`
`{{templateData.key}}`
`{% endraw %}`
`{% endhighlight %}`

It is also possible to parse in user data. The items which can be shown are:
 * `email`
 * `first_name` (Average/ median)
 * `last_name` (Minimum)


# HTML (Data Table, Single Value, List)
It is possible to customize the data table, single value widget and list overview. In order to accomplish this, you need to specify the HTML and CSS. The HTML follows the HTML and Vue JS directives. The css mist be specified by making a JSON setting. You need to configure both. If you want to use data from the query result it is possible. The data is stored inside an object. It can be accessed by using the data in your HTML

## Example (HTML):
```html
<div :style="css.testen">
testeten
</div>
```

## Example:
`{% highlight html%}`
`{% raw %}`
`{{data.key}}`
`{% endraw %}`
`{% endhighlight %}`


## Example (CCS):
```json
{
  "testen": {
    "color": "red",
    "font-size": "40px"
  }
}
```
