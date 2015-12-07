---
layout: spec
title: Specification - Fiscal Data Package
version: 0.3.0-alpha6
updated: 7 December 2015
created: 14 March 2014
author:
 - Tryggvi Björgvinsson (Open Knowledge)
 - Rufus Pollock (Open Knowledge)
 - Paul Walsh (Open Knowledge)
summary: Fiscal Data Package is a lightweight and user-oriented format for publishing and consuming fiscal data. Fiscal data packages are made of simple and universal components. They can be produced from ordinary spreadsheet software and used in any environment.
---

<div class="alert alert-info" markdown="block">
NOTE: This is a draft specification and still under development. If you have comments
or suggestions please file them in the [issue tracker][issues]. If you have
explicit changes please fork the [git repo][repo] and submit a pull request.
</div>

[issues]: https://github.com/openspending/budget-data-package/issues
[repo]: https://github.com/openspending/budget-data-package/issues

# Table of Contents
{:.no_toc}

* Will be replaced with the ToC, excluding the "Contents" header
{:toc}

# Changelog

- `0.3.0-alpha6`: change name of dimension `fields` to `attributes`, reversion of measures and dimension fields to objects
- `0.3.0-alpha5`: variety of improvements and corrections including #35, #37 etc
- `0.3.0-alpha4`: reintroduce a lot of the content of data recommendations from v0.2
- `0.3.0-alpha3`: rework mapping structure in various ways
- `0.3.0-alpha2`: rename Budget Data Package to Fiscal Data Package
- `0.3.0-alpha`: very substantial rework of spec to use "mapping" approach between physical and logical model. Core framework, based on Tabular Data Package, is unchanged.
- [`0.2.0`](./0.2/): large numbers of changes and clarifications for particular fields but no substantive change to the overall spec
- [`0.1.0`](./0.1/): first complete version of the specification

# Overview

Data on government budgets and spending is becoming available in unprecedented quantities. The practice of publishing budget information as machine-readable and openly licensed data is spreading rapidly and will become increasingly standard.

Fiscal Data Package is an open specification for quantitative fiscal data, especially data generated during the planning and execution of budgets. It supports both data on expenditures and revenues, and also supports publishing both highly aggregated and highly granular data, for example individual transactions.

The specification is both simple and easy for publishers to use and, at the same time, sufficiently rich and structured to be useful and processable - especially machine processable - by consumers. In particular, Fiscal Data Packages are:

* Made from lightweight and easily made components (CSV data, JSON metadata)
* Structured according to a simple open standard
* Self-documented with metadata
* Including sufficient information to allow for automated and standardized processing and analysis

Fiscal Data Package specifies the *form* for fiscal data and offers a standardized framework for the *content*. By giving a common *form* to budget data, Fiscal Data Package frees data users from the artificial obstacles created by the lack of a standard structure. By clarifying the *content* of budget data and recommending standard information that fiscal data should contain, Fiscal Data Package helps make data releases more comparable and useful. The Fiscal Data Package specification provides support for:

* A simple and standard way to provide rich metadata about fiscal information - where it came from, who produced it, how it is licensed, what time period it covers etc
* Mapping the raw "physical" model, as represented by columns in the data files, to a standardized "logical" model based around basic fiscal concepts: amounts spent, suppliers, administrative and functional classifications etc
* Progressive enhancement of data via a range of *recommended*, but not *required* metadata, in order to establish a clear path for data providers to enhance data quality, and to address new use cases going forward.


# Background

This proposal assumes some familiarity with fiscal data - e.g. budgets, spending etc - as this is the data we are structuring and describing.

Often, this data takes the form of rows in a spreadsheet or database with each row describing some kind of expenditure or receipt of money. The data can get considerably more complex but keep this simple model in mind for what follows.

```
+--------+------+------------+------------+
| Amount | Year | Department | Spend Type |
+--------+------+------------+------------+
| 1500   | 2014 | Education  |   Capital  |
+--------+------+------------+------------+
| ....
+-----
```

This proposal also builds on and reuses the [Data Package][dp] specifications. These are a family of simple, lightweight formats for publishing data. If you are unfamiliar with these, more information can be found in the Appendix.

# Form and Structure

A Fiscal Data Package has a simple structure:

* Data: the data `MUST` be stored in CSV files.
* Descriptor: there must a descriptor in the form of a single `datapackage.json` file. This file describes both the data and the "package" as a whole (e.g. who created it, its license etc).

Fiscal Data Package builds on the [Data Packages specifications][dp] by defining a "profile" that places some additional constraints on the metadata relevant to describing fiscal data.  A Fiscal Data Package also extends the [Tabular Data Package][tdp] profile (`tabular`) which itself extends the `base` Data Package format.  In this sense, a Fiscal Data Package is a [Tabular Data Package][tdp] which is, in turn, a [Data Package][dp]. We will spell out key implications of this as we proceed.

## File structures

Here are some examples of what a Fiscal Data Package looks like on disk. Usually, the `datapackage.json` and data files are bundled together, and collectively referred to as "the data package".

A simple example of a Fiscal Data Package:

File  | Comment 
------|-------
`datapackage.json`           | Descriptor file
`data/my-financial-data.csv` | Data files, must be .csv. In `data/` by convention, but not required to be.

A more complex example, with additional files:

File  | Comment 
------|-------
`datapackage.json`            | Descriptor file
`README.md`                   | Optional, extra files are ignored.
`data/my-financial-data.csv`  | Actual data, referred to by descriptor.
`archive/my-original-data.xls` | Directory for original sources and "archival" material (optional)
`scripts/scrape-and-clean-the-data.py `| Scripts used in preparing the data package (optional)

And, an example of a data package with normalized data could be:

File  | Comment 
------|-------
`datapackage.json` | Defines foreign key references between the primary file and secondary data files.
`data/my-financial-data.csv`                                | Actually contains spend data. 
`data/my-list-of-entities-receiving-money.csv`              | Data that augmented the spend data, linked by foreign key.
`data/my-list-of-projects-the-money-is-associated-with.csv` | additional augmenting data

## Data files

The data in your Data Package `MUST`:

* Be in CSV format.
* Have well-structured CSVs- no blank rows, columns etc. [Tabular Data Package][tdp] spells this out in detail.

*Note: you can store other data files in your data package - for example, you may want to archive the original xls or data files you used. However, we do not consider these data for the purposes of this specification.*

## The Descriptor - `datapackage.json`

A Fiscal Data Package `MUST` contain a `datapackage.json` - it is the central file in an Fiscal Data Package.

The `datapackage.json` contains information in three key areas:

* Package Metadata - title, author etc
* Resources - describing data files
* Mapping - mapping the source data to a "Logical" model

We will detail each in turn.

## General Package Metadata

This follows [Data Package][dp] (DP). In particular, the `name` and `title` properties `MUST` be on the top-level descriptor:

```javascript
{ 

  // REQUIRED (DataPackage): a url-compatible short name ("slug") for the package
  "name": "australia-2014",

  // REQUIRED (DataPackage): a human readable title for the package
  "title": "Australian Annual Budget 2014",

  // RECOMMENDED (DataPackage): the license for the data in this package.
  "license": "cc-by 3.0",

  // RECOMMENDED: other properties such as description, homepage, version, sources, author, contributors, keywords, as specified in dataprotocols.org/data-packages/

  // RECOMMENDED: a valid 2-digit ISO country code (ISO 3166-1 alpha-2), or, an array of valid ISO codes (if this relates to multiple countries). This field is for listing the country of countries associated to this data.  For example, if this the budget for country then you would put that country's ISO code.
  "countryCode": "AU", // or [ "AU", "NZ" ]

  // RECOMMENDED: the "profile set" for this package. If the `profiles` key is present, it MUST be set to the following object:
  "profiles": {
    "fiscal": "*",
    "tabular": "*"
  },

  // OPTIONAL: a keyword that represents the type of spend data:
  //   * "transaction": rows have dates, and correspond to individual transactions
  //   * "aggregated": rows are summaries of expenditure across a fiscal period
  "granularity": "aggregated", 
  
  // OPTIONAL: the fiscal period of the dataset
  "fiscalPeriod": {
    "start": "1982-04-22",
    "end": "1983-04-21"
  },

  // OPTIONAL: ...other properties...

  // REQUIRED: array of CSV files contained in the package. Defined in http://dataprotocols.org/data-packages/ and http://dataprotocols.org/tabular-data-package/ . Note: 
  //   * Each data file `MUST` have an entry in the `resources` array
  //   * That entry in the `resources` array `MUST` have a JSON table schema describing the data file. (see http://dataprotocols.org/json-table-schema/)

  "resources": [ /* ... */ ],

  // REQUIRED, see "Mapping"
  "mapping": {

    // REQUIRED: array of measures in logical model
    "measures": {
      /* ... */ // REQUIRED at least 1: see "Measures"
    },

    // REQUIRED: array of dimensions in logical model
    "dimensions": {
      /* ... */ // REQUIRED at least 1: see "Dimensions"
    }
  }

}
```

## Mapping

The `mapping` object links columns in the CSV files ("physical model") to pre-defined semantic concepts like transaction dates, amounts, classifications, administrative hierarchies and geographic locations ("logical model").

<img src="https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/pub?w=710&h=357" alt="" />

*Diagram illustrating how the mapping connects the "physical" model (raw CSV files) to the "logical", conceptual, model. The conceptual model is heavily oriented around OLAP.  ([Source on Gdocs](https://docs.google.com/drawings/d/1krRsqOdV_r9VEjzDSliLgmTGcbLhnvd6IH-YDE8BEAY/edit))*
{: style="text-align: center"}

### Measures

Measures correspond to an actual amount in a budget line and are represented by an entry in the `measures` object. Each measure defines the column in the source data which contains this amount. The `measures` object has the following structure:

```javascript
"measures": { 
  "measure-name": {
    // REQUIRED: Field name of source field
    "source": "amount",
    
    // REQUIRED: Any valid ISO 4217 currency code.
    "currency": "USD",
    
    // OPTIONAL: A factor by which to multiple the raw monetary values to get the real monetary amount, eg `1000`. Defaults to `1`.
    "factor": 1,
    
    // OPTIONAL: Resource containing the source field. Defaults to the first resource in the `resources` array.
    "resource": "budget-2014-au",
    
    // OPTIONAL: A keyword that represents the *direction* of the spend, being one of "expenditure" or "revenue".
    "direction": "expenditure",
    
    // OPTIONAL: The phase of the budget that the values in this measure relate to. 
    // It `MUST` be one of the following strings: proposed, approved, adjusted, executed
    "phase": "proposed",
    
    // OPTIONAL: Other properties allowed.
  }
  //...
}
```

### Dimensions

Dimensions provide the "context" for a measure and are represented by entries in the `dimensions` object.  Each `dimension` can have multiple attributes, and each attribute can have multiple fields.  Each field defines the column in the source data which contains an element of the contextual information for the spending in the budget line.  The `dimensions` object has the following structure:

```javascript
"dimensions": {
  "dimension-name": {
    // REQUIRED: An array of attribute objects that make up the dimension. Each field of an attribute must have either a `source` key referencing the field name in the resource from which the data is mapped or a `constant` key that provides its value.
    "attributes": [
      {
        "title": {    
          // EITHER: the field name of the source file from which the value is derived for this property
          "source": "project_field_on_source",
          // OR: a single value that applies for all rows of the dataset.
          "constant": "Some Project",

          // OPTIONAL: the resource in which the field is located. Defaults to the first resource in the `resources` array.
          "resource": "budget-2014-au"
        },
        "id": {
          "source": "class_code"
        }
      }
    ],
    
    // REQUIRED: Either an array of strings corresponding to a set of field names (keys within attribute objects) or a single string referencing one of these. The value of `primaryKey` indicates the primary key or primary keys for the dimension.
    "primaryKey": ["id"],

    // OPTIONAL: Describes what kind of a dimension it is. `dimensionType` is a string that `MUST` be one of the following:
    // * "datetime": the date of a transaction 
    // * "entity": names the organisation doing the spending or receiving
    // * "classification": one or more fields that create a categorical hierarchy of the type of spending (eg, Health > Hospital services > Nursing). Combine with `classificationType` for greater expressiveness.
    // * "activity": names a specific programme or project under which the money is spent
    // * "fact": an attribute such as an ID or reference number attached to a transaction
    // * "location": the geographical location where money is spent
    // * "other": not one of the above
    "dimensionType": "classification",

    // RECOMMENDED (if using dimensionType="classification"). The basis on which transactions are being classified, one of these values:
    // * "administrative": an organisational structure, such as Portfolio > Department > Branch
    // * "functional": the purpose of the spending, such as Health > Hospital services > Nursing
    // * "economic": focused on the nature of the accounting, such as Compensation > Wages and salaries > Wages and salaries in cash
    "classificationType": "administrative"

    // OPTIONAL: Other properties allowed.

  }
  //...
]
```

## Examples

{% assign sorted_pages = site.pages | sort:"order" %}
{% for page in sorted_pages %}
  {% if page.category == 'example' %}
  * [{{ page.title }}]({{ page.url | remove: 'index.html' }}) 
  {% endif %}
{% endfor %}

# Content

This section provides a standard framework for the "content" of Fiscal Data Packages. The previous section has been about the form both for the data (e.g. that it is CSV) and for the metadata (the information in the datapackage.json). This section is about the "content", that is the kind of actual data a Package contains. In particular, it sets out guidelines for what information, exactly, is present. For example, that government budget information is classified according to a standard classification codesheet like the [UN's COFOG][cofog].

[cofog]: http://data.okfn.org/data/core/cofog/

Content requirements will necessarily vary across the different types of fiscal data. For example, the data describing high level budgets may be different from that describing day-to-day expenditures, and expenditure information may be different from revenue. Thus, our framework will distinguish different types of fiscal data.

We also emphasize that what we provide is a framework rather than a strict standard. That is, we provide recommendations on what information should be provided rather than strict requirements. These recommendations are also categorised into "quality" levels. Each level requires more information be provided.

Finally, our recommendations place requirements on the "logical" model not the physical model. Of course, the logical model data is sourced from the physical model so requirements on the logical model ultimately place requirements on the physical model. However, by defining our requirements on the logical model, we keep the flexibility in naming and structure of the raw, source data - for example, whilst a classification dimension with COFOG data should be named `cofog` and have a field called `code` your source CSV could have that COFOG data in a column called "COFOG-Code" or "Classification" or any other name.


## Required data (all categories)

All datasets MUST have at least one measure. Essentially this is requiring each dataset have at least one column which corresponds to an "amount" of money.

## Recommended data (all categories)

The following fields SHOULD be included wherever possible:

* `id`: A globally unique identifier for the budget item. This `id` field will usually be located on the default `fact` dimension.

## Special Dimensions

### Classifications

It is common for fiscal data to be classified in various ways. A classification is a labelling of a given item with a reference to standardized codesheet.

Classifications will be represented in the mapping as a dimension. Each classification dimension `MUST` have a `code` field whose value will correspond to the classification code in the official codesheet. Sometimes classifications can change and we recommend utilizing a `version` field if there is a need to indicate the version of a classification.

Whenever we have a code field in a classification dimension, the licit values for that field consist of the numerical codes from the appropriate codesheet, with hierarchical levels separated by periods. `1.1.4.1.3` is a licit value for a dimension named `gfsm`, for example, corresponding to the code for "Turnover and other general taxes on goods and services".

Classifications are of different types. The type of the classification `MAY` be indicated using the `classificationType` attribute on the dimension. Values are:

* `functional`
* `administrative`
* `economic`

It is common for classifications to be hierarchical and have different levels. If this is present in your data and you wish to record it in the mapping, we recommend creating an entry in the attributes array for each level of the hierarchy.  Hierarchical order will be inferred by the attribute's position in the array:

```
"your-classification": {
  "attributes": [
    {
      "level1": {
        "source": "..."
      }
    },
    {
      "level2": {
        "source": "..."
      }
    }
  ]
}
```

#### COFOG (Classifications of Functions of Government)

This classification uses the United Nations [Classification of the Functions of Government][cofog]. Here is the simplest example as a dimension:

```
"cofog": {
  "attributes": [
    {
      "code": {
        "source": "..."
      }
    }
  ]
}
```

#### IMF GFSM

GFSM is the [IMF Government Finance Statistics Manual (2014)][gfsm2014]. For expenditure classification, use Table 6.1. For revenue, use Table 5.1.

[gfsm2014]: http://www.imf.org/external/np/sta/gfsm/

#### Chart of Accounts

It is common for both revenue and expenditure that there is some general "economic" classification for the item using the publisher's chart of accounts. Relevant fields for this dimension:

* `code`:  The internal code identifier for the economic classification.
* `title`:  Human-readable name of the economic classification of the budget item (i.e. the type of expenditure, e.g. purchases of goods, personnel expenses, etc.), drawn from the publisher's chart of accounts.

### Programs and Projects

Expenditures are frequently associated with a program or project. Often these terms are used interchangeably. There is a rough distinction:

* Program: A program is a set of goal-oriented activities, such as projects, that has been reified by the government and made the responsibility of some ministry. A program can, e.g. be a government commitment to reducing unemployment.
* Project: A project is an indivisible activity with a dedicated budget and fixed schedule. A project can be a part of a bigger program and can include multiple smaller tasks. A project in an unemployment reduction program can e.g. be increased education opportunities for adults.

In terms of representation as a dimension, we use a `dimensionType` of "activity".  The structure is as follows:

```
"your-chosen-program-name": {
  "dimensionType": "activity",
  "attributes": [
    {
      "title": {
        # Name of the government program or project underwriting the budget item.

        "source": "..."
      },
      "id": {
        # The internal code identifier for the government program or project
        
        "source": "..."
      }
    }
  ]
}
```

### Entities

An entity is a distinct organization, government department, or individual that is spending or receiving a given amount.  Entities will be represented by dimensions.

#### Administrators

* `title`: The title or name of the government entity legally responsible for spending the budgeted amount.
* `id`: The internal code for the administrative entity.
* `location`: Reference to a `location` dimension listing geographical region where administrative entity is located.
* `locationCode`: code for geographical region where administrative entity is located.

#### Accounts

Whilst strictly not an entity, the concept of an "account" from which money is spent or into which money is deposited is common and closely resembles an "entity" in terms of functionality within fiscal analysis (e.g. tracing the movement of money between different pots). Recommended fields on an account dimension are:

* `title`: The fund into which the revenue item will be deposited. (This refers to a named revenue stream.)
* `id`: The internal code identifier for the fund.

### Location

There is a frequent desire to label items with location, usually by attaching geographic codes for a region or area. This allows the spending or revenue to  be analysed by region or area. This geographic information can be introduced directly by classifying the item with a code, or, more frequently indirectly by associating a geographic code to e.g. an entity. For example, by labelling a supplier with their location one can then associate a spend with that supplier as spending in that location.

We RECOMMEND using a `location` dimension though fields may also be applied directly onto another object (e.g. an entity). Here are fields that `MAY` be applied either directly to an item or to an entity or other object associated to an item.

* `code`: The internal or local geographicCode id based for the geographical region
* `title`: Name or title of the geographical region targeted by the budget item
* `codeList`: the geo codelist from which the geocode is drawn
* `ocdid`: The [Open Civic Data Division Identifier](http://docs.opencivicdata.org/en/latest/proposals/0002.html), if it exists, for the geographical region

Note when applying these as fields directly on an object we suggest prefixing each value with `geo` so you would have `geoCode` rather than `code` etc.

----

## Suggested Dimensions for Different Types of Spending Data

This section lists the suggested sets of dimensions that can usefully describe different types of spending data.  We also include quality levels for each dimension.  For example, if your aggregated expenditure data has a dimension named "cofog" described as above, it will have a quality level of "2".  Quality levels in parentheses are applied to dimensions which are desirable but not required to reach that quality level.

### Aggregated expenditure data

Aggregated expenditure data (direction `expenditure`, granularity `aggregated`) describes planned or executed government expenditures. These planned expenditures are disaggregated to at least the *functional category* level, and they can optionally be disaggregated up to the level of individual projects.

Aggregated data is in many cases the proposed, approved or adjusted budget (but can also be an aggregated version of actual expenditure). For this reason there are fields in aggregated data which are not applicable to transactional data, and vice versa.

| Dimension | Type | Quality | Description|
| ----- | -------- | ------- | ---------- |
| cofog | `classification` | 2 | The COFOG functional classification for the budget item. |
| gfsm  | `classification` | 2 | The GFSM 2014 economic classification for the budget item. |
| chart-of-accounts | `classification` | 2 | Human-readable name of the (non-COFOG) functional classification of the budget item (i.e. the socioeconomic objective or policy goal of the spending; e.g. "secondary education"), drawn from the publisher's chart of account. |
| administrator | `entity` | 2 | The name of the government entity legally responsible for spending the budgeted amount. |
| account | `entity` | 3 | The fund from which the budget item will be drawn. (This refers to a named revenue stream.) |
| program | `activity` | 3 |  Name of the government program underwriting the budget item. |
| procurer | `entity` | (3) | The government entity acting as the procurer for the transaction, if different from the institution controlling the project. |

### Aggregated revenue data

Aggregated revenue data (direction `revenue`, granularity `aggregated`) describes projected or actual government revenues, disaggregated to the *economic category* level.

Aggregated data is in many cases the proposed, approved or adjusted budget (but can also be an aggregated version of actual revenue). For this reason there are fields in aggregated data which are not applicable to transactional data, and vice versa.

| Dimension | Type | Quality | Description|
| --------- | ---- | ------- |------------|
| chart-of-accounts | `classification` | (2) | Name of the economic classification of the revenue item, drawn from the publisher's chart of account. |
| gfsm | `classification` | 2 | The GFSM 2014 economic classification for the revenue item. |
| account | `entity` | 3 | The fund into which the revenue item will be deposited. (This refers to a named revenue stream.) |
| recipient | `entity` | 2 | The recipient (if any) targetted by the revenue item. |
| source | `location` | (3) | Geographical region from which the revenue item originates. |

### Transactional expenditure data

Transactional expenditure data (direction `expenditure`, granularity `transactional`) describes government expenditures at the level of individual transactions, exchanges of funds taking place at a specific time between two entities. 

| Dimension | Type | Quality | Description|
| --------- | ---- | ------- | ---------- |
| administrator | `entity` | 2 | The government entity responsible for spending the amount. |
| date | `datetime` | 1 | The date on which the transaction took place. |
| supplier | `entity` | 2 | The recipient of the expenditure. |
| contract | `activity` | 3 | The contract associated with the transaction. |
| budgetLineItem | `fact` | (3) | The unique ID of budget line item (value of id column for budget line) authorizing the expenditure. The budget line can either come from an approved or adjusted budget, depending on if the transaction takes place after the related budget item has been adjusted or not. |
| invoiceID | `fact` | (3) | The invoice number given by the vendor or supplier. |
| procurer | `entity` | (3) | The government entity acting as procurer for the transaction, if different from the institution controlling the project. |


# Acknowledgements

Thanks to Vitor Baptista, Sarah Bird, Samidh Chakrabarti, Pierre Chrzanowski, Andrew Clarke, Velichka Dimitrova, Friedrich Lindenberg, James McKinney, Rufus Pollock, Paolo de Renzio, Martin Tisne and Paul Walsh.

----

# Appendix

## Budget data

A budget is over a year-long process of planning, execution, and oversight of a government's expenditures and revenues. At multiple stages in the process, *quantitative data* is generated, data which specifies the sums of money spent or collected by the government. This data can represent either plans/projections or actual transactions.

In a typical budget process, a government authority, e.g. the executive arm, will put together a **proposed** budget and submit that for approval, e.g. by the country's legislative arm. The approval process might involve making changes to the proposal before the **approved** version is accepted. As time goes by there is a possibility that some projects, institutions etc. will require more money to fulfill their task so adjustments need to be made to the approved budget. The **adjustment** is then approved by the original budget entity, e.g. the legislative arm. This usually requires reasoning for why the original budget was not sufficient. The **executed** budget is the actual money spent or collected which can then be compared to the approved and adjusted plan.

By recognizing the following distinctions between data types, Fiscal Data Package is expressive enough to cover the data generated at every stage:

* Data can represent either *expenditures* or *revenues*.
* Data can be either *aggregated* or *transactional*. An item of aggregated data represents a whole category of spending (e.g. spending on primary education). An item of transactional data represents a single transaction at some specific point in time.
* Data can come from any stage in the budget cycle (proposal, approval, adjustment, execution). This includes three different types of planned / projected budget items (proposal, approval, adjustment) and one representing actual completed transactions (execution).

### Budget hierarchy and categorizations

Budget data has various degrees of hierarchy, depending on the perspective. From a functional perspective it can use a functional classification. The functional classification can be set up as a few levels (a hierarchy). An economical classification is not compatible with the functional hierarchy and has a different hierarchy. Another possible hierarchy would be a program/project hierarchy where many projects are a part of a program.

All of these hierarchies give a picture of how the budget line fits into the bigger picture, but none of them can give the whole picture. Budget data usually only includes general classification categories or the top few hierarchies. For example a project can usually be broken down into tasks, but budget data usually would not go into so much detail. It might not even be divided into projects.

Categorizing and organizing the data is more about describing it from the bigger perspective than breaking it down into detailed components and the Fiscal Data Package specification tries to take that into account by including top level hierarchies and generalised classification systems but there is still a possibility to go into details by supplying a good description of every row in the budget data.



[dp]: http://dataprotocols.org/data-packages/
[tdp]: http://dataprotocols.org/tabular-data-package/
[bdp]: https://github.com/openspending/budget-data-package
[bdp-resources]: https://github.com/openspending/budget-data-package/blob/master/specification.md#budget-specific-metadata
[dp-profiles]: https://github.com/dataprotocols/dataprotocols/issues/87
[dp-resources]: http://dataprotocols.org/data-packages/#resource-information
[fd]: http://data.okfn.org
[mapping]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[views]: http://docs.openspending.org/en/latest/model/design.html#views-and-pre-defined-visualizations
[jts]: http://dataprotocols.org/json-table-schema/
[current-model]: http://docs.openspending.org/en/latest/model/design.html
[cofog]: http://unstats.un.org/unsd/cr/registry/regcst.asp?Cl=4
[imf-budget]: http://www.imf.org/external/pubs/ft/tnm/2009/tnm0906.pdf
[olap]: https://en.wikipedia.org/wiki/Online_analytical_processing
