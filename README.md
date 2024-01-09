Introduction
------------

The Data and Analytics section (D&A) of UNICEF collects, integrates, analyzes, models, and publishes hundreds of indicators on the state of children and women at global, regional, national, and subnational level. Owing to the wide variety of its data that span many thematic areas it has historically found it challenging to house these data in a central location that is easy to discover, query, and consume.

The BMGF-funded Helix project enhances UNICEF’s ability to co-locate and make these public indicators available using a single set of services that conform to the UN-preferred Statistical Data and Metadata eXchange standard (SDMX). In addition, the Helix project works to integrate the business process to deliver consistent and coordinated data.

This project aims at making searches of indicator values convenient, removing some of the query limitations imposed by SDMX, the standard used to exchange statistical data within the Helix project. Youyou can access an Apache Solr API to answer many additional questions, like retrieving all values of indicators with a specific value on a dimension, filtering results by a subset of domains, retrieving all values of indicators related to a specific data source, etc.

The Solr index is available [here](https://solr-helix.unicef.org/solr)

Guests can query the index using the credentials:

`Username: solr`

`Password: solr`

This user can query the index, but it has no write permissions.

Structure of the index
----------------------

The index is composed of a set of objects, each one representing a specific indicator of a country in a year. The structure is hierarchical since an indicator is composed of a set of Disaggregations, each one composed of Attributes and Dimensions. There are properties (in Solr they are called **fields**; “property” is a word coming from object-oriented programming) at the level of the ROOT of the indicator (like country, country code, year, total value, data source of total value, etc.) and properties at the level of each disaggregation (like value and data source for that specific disaggregation). Dimensions and Attributes are highly flexible, and they are represented as triples Name, Value, Code, so they are not hardcoded in the index. Still, we can always have new dimensions or attributes and the schema does not change.

About the data source, in SDMX sometimes it is modeled as a dimension, sometimes as an attribute. To make things easier and to enable more powerful queries, in Solr the data source is a property of a disaggregation. In addition, since the ROOT element contains also properties related to the total value of the indicator, the data source related to the total value is repeated there (when the total value is available).

**A clarification**: the "total value" is the value when all dimensions are “Total”.

As an example of a record (in JSON):

             "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014",
             "HelixCode": "PT_ADLS_10-14_LBR_HC",
             "Indicator": "Percentage of adolescents (aged 10-14 years) engaged in household chores",
             "Year": "2014",
             "CountryCode": "AFG",
             "Country": "Afghanistan",
             "content_type": "indicator",
             "TotalValue": null,
             "DATA_SOURCE": null,
             "Disaggregations": [
                 {
                 "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T",
                 "TotalValue": "9.3",
                 "DATA_SOURCE": "Living Conditions Survey 2013-14",
                 "Dimensions": [
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|SEX",
                         "Name": "SEX",
                         "Value": "Male",
                         "Code": "M"
                     },
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|AGE",
                         "Name": "AGE",
                         "Value": "10 to 14 years old",
                         "Code": "Y10T14"
                     },
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|EDUCATION_LEVEL",
                         "Name": "EDUCATION_LEVEL",
                         "Value": "Total",
                         "Code": "_T"
                     },
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|WEALTH_QUINTILE",
                         "Name": "WEALTH_QUINTILE",
                         "Value": "Total",
                         "Code": "_T"
                     },
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|RESIDENCE",
                         "Name": "RESIDENCE",
                         "Value": "Total",
                         "Code": "_T"
                     },
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|NUMBER_CHLD",
                         "Name": "NUMBER_CHLD",
                         "Value": "Total",
                         "Code": "_T"
                     }
                 ],
                 "Attributes": [
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|UnitOfMeasure",
                         "Name": "UnitOfMeasure",
                         "Value": "%",
                         "Code": "PCNT"
                     },
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|TimePeriodMethod",
                         "Name": "TimePeriodMethod",
                         "Value": "End of fieldwork"
                     },
                     {
                         "ID": "UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|Coverage",
                         "Name": "Coverage",
                         "Value": "2013-14"
                     }
                 ]
             }]
        

How to query Solr (examples)
----------------------------

Apache Solr has strong support for nested documents. To enable nested documents, all children must have a unique identifier. In our case, children's identifiers are an extension of the parent's identifiers. So, for example, the attribute with ID:  
`"UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T|UnitOfMeasure"`  
belongs to the disaggregation with ID:  
`"UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014|M|Y10T14|_T|_T|_T|_T"`  
which belongs to the indicator:  
`"UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014"`.

Solr keeps track of the hierarchy in a field called _\_nestpath_. In our index, the hierarchy is expressed in this way:

     _nest_path_:"/Disaggregations"  
     _nest_path_:"/Disaggregations/Attributes"  
     _nest_path_:"/Disaggregations/Dimensions"

When you query Solr, you can use the field `q` to type your query. In addition, the field `fl` can be used to retrieve specific fields of documents that match the query. For example, if you are matching a ROOT document and you want to see the whole hierarchy, `fl` must be `*,[child]`. By default, only 10 children are displayed. If your document has more than 10 children, you can change this parameter using the "limit" option: `*,[child limit=100]` Other combinations are possible, also filtering on fields of children. Solr has a very powerful tool that allows doing a lot of things with hierarchies, for example matching children and retrieving parents. It’s the Block Join Query Parser. You can read [this article](https://sease.io/2019/06/apache-solr-childfilter-transformer.html) if you want to learn more about the queries that you can build.

<a name="index_examples"/>
The rest of this document provides some examples of queries:

*   [Retrieve an indicator by ID with all the disaggregations](#retrieve-an-indicator-by-id-with-all-disaggregations)
*   [Retrieve all indicators from a given country in a given year](#retrieve-an-indicator-by-country)
*   [Filter by one or more domains](#filter-by-a-subset-of-domains)
*   [Get all the disaggregations whose dimension WEALTH\_QUINTILE is LOWEST](#get-all-disaggregations-whose-dimension-wealth_quintile-is-lowest)
*   [Limit the output to some children's fields](#limit-output-to-some-children-fields)
*   [Limit the output to a subset of domains](#limit-output-to-a-subset-of-domains)
*   [Limit the output to specific parent fields](#limit-output-to-specific-parent-fields)
*   [Query on the code of a dimension (not on the value)](#query-on-the-code-of-a-dimension-and-not-the-value)
*   [Get all WEALTH\_QUINTILE dimensions available in the index](#get-all-wealth_quintile-dimensions-available-in-the-index)
*   [Filter on data sources](#filter-on-data-sources)
*   [Get the most recent version of an indicator](#get-the-most-recent-version-of-an-indicator)

### Retrieve an indicator by ID with all the disaggregations
<a name="retrieve-an-indicator-by-id-with-all-disaggregations"/>

[Back to all examples](#index_examples)

The ID of an indicator is the indicator identifier available in Solr, e.g. `"UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|AFG|2014"`. It is not the indicator Helix Code.

q

`ID:"UNICEF|PT|1.0|PT_ADLS_10-14_LBR_HC|COD|2014"`

fl

`*,[child limit=50]`

Remove the `fl` filter to retrieve only the ROOT, so the indicator with total value and without disaggregations.

[Link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=ID%3A%22UNICEF%7CPT%7C1.0%7CPT_ADLS_10-14_LBR_HC%7CCOD%7C2014%22)

### Retrieve all indicators from a given country in a given year
<a name="retrieve-an-indicator-by-country"/>

[Back to all examples](#index_examples)

q

`+content_type:"indicator" +CountryCode:"COD" +Year:2013`

fl

`HelixCode`

[Link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=%2Bcontent_type%3A%22indicator%22%20%2BCountryCode%3A%22COD%22%20%2BYear%3A2013)

### Filter by one or more domains
<a name="filter-by-a-subset-of-domains"/>

[Back to all examples](#index_examples)

Domains are encoded in indicators' IDs. Thus, whatever query you are writing, you can add a filter that limits indicators to specific domains. For example, to filter only by Nutrition and Child Protection:  
`Existing Query AND (ID:(UNICEF|PT*) OR ID:(UNICEF|NT*)) AND content_type: "indicator"`

`Content type` filter is needed if you want to match ROOT documents, i.e. indicators.

### Get all the disaggregations whose dimension WEALTH\_QUINTILE is LOWEST
<a name="get-all-disaggregations-whose-dimension-wealth_quintile-is-lowest"/>

[Back to all examples](#index_examples)

q

`{!parent which='_nest_path_:"/Disaggregations"'}+Value:"Lowest" +Name:"WEALTH_QUINTILE"`

fl

`*,[child limit=50]`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=%7B!parent%20which%3D%27_nest_path_%3A%22%2FDisaggregations%22%27%7D%2BValue%3A%22Lowest%22%20%2BName%3A%22WEALTH_QUINTILE%22)

This query searches the Value “Lowest” in all Dimensions with name “WEALTH\_QUINTILE” and returns all Disaggregation (the parent filter with the specification of the nest path of the hierarchy) with all Dimensions and Attributes (the `fl` filter). If you prefer to search by code, you can search for "Q1", which is the code for "Lowest":

q

`{!parent which='_nest_path_:"/Disaggregations"'}+Code:"Q1" +Name:"WEALTH_QUINTILE"`

fl

`*,[child limit=50]`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=%7B!parent%20which%3D%27_nest_path_%3A%22%2FDisaggregations%22%27%7D%2BCode%3A%22Q1%22%20%2BName%3A%22WEALTH_QUINTILE%22)

### Limit the output to some children's fields
<a name="index_examples"/>

[Back to all examples](#limit-output-to-some-children-fields)

If we want to limit the returned hierarchy only to the dimension wealth quintile, we can express FL as:

fl

`*,[child childFilter=Name:WEALTH_QUINTILE]`

In this case, the output contains all the disaggregations matching the query with all the properties (indicator value and data source), but without attributes and with the only dimension "wealth quintile".

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20childFilter%3DName%3AWEALTH_QUINTILE%5D&q=%7B!parent%20which%3D%27_nest_path_%3A%22%2FDisaggregations%22%27%7D%2BValue%3A%22Lowest%22%20%2BName%3A%22WEALTH_QUINTILE%22)

### Limit the output to a subset of domains
<a name="limit-output-to-a-subset-of-domains"/>

[Back to all examples](#index_examples)

q

`{!parent which='_nest_path_:"/Disaggregations" AND (ID:(UNICEF|PT*) OR ID:(UNICEF|NT*))'}+Value:"Lowest" +Name:"WEALTH_QUINTILE"`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C[child]&q={!parent which%3D'_nest_path_%3A%22%2FDisaggregations%22 AND (ID%3A(UNICEF|PT*) OR ID%3A(UNICEF|NT*))'}%2BValue%3A%22Lowest%22 %2BName%3A%22WEALTH_QUINTILE%22)

### Limit the output to specific parent fields
<a name="limit-output-to-specific-parent-fields"/>

[Back to all examples](#index_examples)

If in output we want to see only the total value and the disaggregation ID, we can use FL to filter only on that attribute:

fl

`ID,TotalValue`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=TotalValue%2CID&q=%7B!parent%20which%3D%27_nest_path_%3A%22%2FDisaggregations%22%20AND%20(ID%3A(UNICEF%7CPT*)%20OR%20ID%3A(UNICEF%7CNT*))%27%7D%2BValue%3A%22Lowest%22%20%2BName%3A%22WEALTH_QUINTILE%22)

### Query on the code of a dimension (not on the value)
<a name="query-on-the-code-of-a-dimension-and-not-the-value"/>

[Back to all examples](#index_examples)

In case we don’t want to use the value “Lowest”, but the code “Q1”, the query becomes:

q

`{!parent which='_nest_path_:"/Disaggregations"'}+Code:"Q1" +Name:"WEALTH_QUINTILE"`

fl

`*,[child limit=50]`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=%7B!parent%20which%3D%27_nest_path_%3A%22%2FDisaggregations%22%27%7D%2BCode%3A%22Q1%22%20%2BName%3A%22WEALTH_QUINTILE%22)

### Get all WEALTH\_QUINTILE dimensions available in the index
<a name="get-all-wealth_quintile-dimensions-available-in-the-index"/>

[Back to all examples](#index_examples)

q

`+_nest_path_:"/Disaggregations/Dimensions" +Name:"WEALTH_QUINTILE"`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?q=%2B_nest_path_%3A%22%2FDisaggregations%2FDimensions%22%20%2BName%3A%22WEALTH_QUINTILE%22)

### Filter on data sources
<a name="filter-on-data-sources"/>

[Back to all examples](#index_examples)

The Data Source is available at the level of disaggregations. This is important because, if two data sources provide a different value for the same combination of dimensions of an indicator, we maintain both values in the index. To get all disaggregations (with all Attributes and Dimensions) of a specific data source:

q

`+_nest_path_:"/Disaggregations" +DATA_SOURCE:"Living Conditions Survey 2013-14"`

fl

`*,[child limit=50]`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=%2B_nest_path_%3A%22%2FDisaggregations%22%20%2BDATA_SOURCE%3A%22Living%20Conditions%20Survey%202013-14%22)

When disaggregations are not available but the total value is available, the data source is represented as an attribute of the ROOT element, together with the value. To discover all indicators having a data source for their total value, we can query Solr in two ways:

q

`-_nest_path_:* +DATA_SOURCE:*`

q

`+content_type:"indicator" +DATA_SOURCE:*`

And then always:

fl

`*,[child limit=50]`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?q=%2Bcontent_type%3A%22indicator%22%20%2BDATA_SOURCE%3A*)

If we want to match a specific data source about indicators' total values:

q

`+content_type:"indicator" +DATA_SOURCE:"TERCE 2013"`

fl

`*,[child]`

If a data source provides values both at disaggregation level and non-disaggregated values, we can combine two queries:

q

`(+content_type:"indicator" +DATA_SOURCE:"TERCE 2013") OR (+_nest_path_:"/Disaggregations" +DATA_SOURCE:"TERCE 2013")`

fl

`*,[child limit=50]`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=(%2Bcontent_type%3A%22indicator%22%20%2BDATA_SOURCE%3A%22TERCE%202013%22)%20OR%20(%2B_nest_path_%3A%22%2FDisaggregations%22%20%2BDATA_SOURCE%3A%22TERCE%202013%22))

The previous query is just an example since in the current index there are no similar situations at this time. In case there will be, results will be a mix of disaggregations (when the data source provides values at disaggregation level) and ROOT elements (when the data source does not provide any disaggregation)

### Get the most recent version of an indicator
<a name="get-the-most-recent-version-of-an-indicator"/>

[Back to all examples](#index_examples)

This query is very useful to access the latest value of an indicator in a country. Thus, in Q we filter by Helix Code and Country Code. Then we sort by year descending and we take only one element in the output:

q

`+CountryCode:"COD" +HelixCode:"PT_ADLS_10-14_LBR_HC"`

fl

`*,[child limit=50]`

sort

`Year desc`

rows

`1`

[link](https://solr-helix.unicef.org/solr/IndicatorsData/select?fl=*%2C%5Bchild%20limit=50%5D&q=%2BCountryCode%3A%22COD%22%2BHelixCode%3A%22PT_ADLS_10-14_LBR_HC%22&rows=1&sort=Year%20desc&start=0)
