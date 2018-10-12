# Meplato Store 2.0 API for ABAP

This is the ABAP client for the Meplato Store 2 API. It helps you write
software to integrate into the Meplato suite for suppliers.

## Prerequisites

You need at two things to use the Meplato Store 2 API.

1. A login to Meplato Store 2.
2. An API token.

Get your login by contacting Meplato Supplier Network Services. The API token
is required to securely communicate with the Meplato Store 2 API. You can
find it in the personalization section when logged into Meplato Store.

## Installation

### Importing the transport requests
The ABAP client for the Meplato Store API comes as a series of transport requests. Please find the current version in the [Downloads](https://github.com/meplato/store2-abap-client#downloads) section.

The development objects included in the transport are in the namespace /WAME/. Currently, the objects are bundled in two packages: /WAME/MSA and /WAME/VS.

Please import the transport requests in your SAP ABAP systems from which you would like to access the Meplato Store API.

### Importing the SSL certificate
The API services are addressed via https. Your SAP systems needs to trust the SSL certificate of the Meplato servers. Please find the certificate file in the [Downloads](https://github.com/meplato/store2-abap-client#downloads) section.

Go to transaction STRUST:
- Certificate
- Import:  
![STRUST Certificate Import](https://github.com/meplato/store2-abap-client/blob/master/files/STRUST_Certificate_Import.jpg)
- File
- Select the certificate:  
![STRUST Certificate Import File](https://github.com/meplato/store2-abap-client/blob/master/files/STRUST_Certificate_Import_File.jpg)
- Execute (the certificate is displayed at the bottom):  
![STRUST Certificate](https://github.com/meplato/store2-abap-client/blob/master/files/STRUST_Certificate.jpg)
- Add the certificate to the identity "SSL Client (Standard)"\* (Add to certificate list):  
![STRUST Certificate Add to List](https://github.com/meplato/store2-abap-client/blob/master/files/STRUST_Certificate_Add_to_List.jpg)
- Save.

\* You might as well use any other client identity if it suits your requirements better (see section [Creating a client](https://github.com/meplato/store2-abap-client#creating-a-client)).


## Getting started

All functionality in the client is separated into services. So you first need to create and initialize a service, then set its parameters, and finally execute it. Here's an example of how to retrieve the list of catalogs for your Meplato Store account.
```abap
DATA CATALOGS_SERVICE TYPE REF TO /WAME/IF_MSA_CATALOGS_SERVICE.
DATA CATALOGSEARCHRESP TYPE REF TO /WAME/IF_MSA_CATALOGSEARCHRESP.
DATA ITEMS TYPE /WAME/IF_MSA_CATALOG=>TT_SELF.
DATA ITEM LIKE LINE OF ITEMS.
DATA CLIENT TYPE REF TO /WAME/IF_MSA_CLIENT.

* Create a client and set your API token
CLIENT = /WAME/CL_MSA_FACTORY=>CREATE_SIMPLE_CLIENT(
             IV_USER           = `<your API token>`
         ).
* Create the Catalogs service
CATALOGS_SERVICE = /WAME/CL_MSA_FACTORY=>CREATE_CATALOGS_SERVICE(
                       IO_CLIENT         = CLIENT
                   ).

* Execute a search request
CATALOGSEARCHRESP = CATALOGS_SERVICE->SEARCH( IV_SKIP = 0 IV_TAKE = 10
                                              IV_SORT = |-{ /WAME/IF_MSA_CATALOG=>E_SERIAL_NAMES-CREATED },{ /WAME/IF_MSA_CATALOG=>E_SERIAL_NAMES-NAME }| " '-created,name'
                                            ).

* Loop at the results
WRITE / |You have { CATALOGSEARCHRESP->GET_TOTALITEMS( ) } catalogs|.
ITEMS = CATALOGSEARCHRESP->GET_ITEMS( ).
LOOP AT ITEMS INTO ITEM.
  WRITE / |Catalog with ID={ ITEM->GET_ID( ) } has name { ITEM->GET_NAME( ) }|.
ENDLOOP.
```
Please note: The ABAP client is generally defined by interfaces. You can create object instances implementing an interface by calling the appropriate `CREATE_...` method of the factory class `/WAME/CL_MSA_FACTORY`.

Let's have a closer look at the individual steps:

### Creating a client

A client is responsible for the communication on a technical level. When creating a service you need to pass on a client which is then used by the service to address the REST endpoints.

Two clients are available:
- The Simple Client (interface `/WAME/IF_MSA_SIMPLE_CLIENT`)
- The Destination Client (interface `/WAME/IF_MSA_DEST_CLIENT`)

#### The Simple Client

When creating a Simple Client, you have to set your API token (as user). Additionally, you might pass on Proxy Settings and an SSL ID:
```abap
DATA SIMPLE_CLIENT  TYPE REF TO /WAME/IF_MSA_SIMPLE_CLIENT.
DATA PROXY_SETTINGS TYPE REF TO /WAME/IF_MSA_PROXY_SETTINGS.

PROXY_SETTINGS = /WAME/CL_MSA_FACTORY=>CREATE_PROXY_SETTINGS(
                     IV_HOST     = `<your proxy host>` "mandatory
*                     IV_SERVICE  = `<your proxy service>` "optional
*                     IV_USER     = `<your proxy user>` "optional
*                     IV_PASSWORD = `<your proxy password` "optional
                 ).

SIMPLE_CLIENT = /WAME/CL_MSA_FACTORY=>CREATE_SIMPLE_CLIENT(
             IV_USER           = `your API token` "mandatory
*             IV_SSL_ID         = '<...>' "optional
*             IO_PROXY_SETTINGS = PROXY_SETTINGS "optional
         ).
```
The API token is actually transmitted as user during authentication. If you need to communicate via a proxy, pass on proxy settings. Set an SSL ID, if you wish to use an SSL ID different from the standard SSL client (cf. [Importing the SSL certificate](https://github.com/meplato/store2-abap-client#importing-the-ssl-certificate)).

#### The Destination Client

When creating a Destination Client, you only need to set an RFC destination of type G. All settings like proxy, API token (user), SSL ID etc. are taken from the destination:
```abap
DATA DESTINATION_CLIENT TYPE REF TO /WAME/IF_MSA_DEST_CLIENT.

DESTINATION_CLIENT = /WAME/CL_MSA_FACTORY=>CREATE_DEST_CLIENT(
                       IV_DESTINATION = '<your RFC destination>' "mandatory
                     ).
```
In the RFC destination...
- set the Target Host to [store.meplato.com](store.meplato.com). You might enter proxy settings if required:  
![RFC Technical Settings](https://github.com/meplato/store2-abap-client/blob/master/files/RFC_Technical_Settings.jpg)
- set the User to your API token and activate SSL. A password is currently not required but please set a dummy one (e.g. a single letter) such that the PW Status changes to "saved" (otherwise the user won't be transmitted). You might set the SSL Certificate to another SSL ID than "SSL Client (Standard)" if required (cf. [Importing the SSL certificate](https://github.com/meplato/store2-abap-client#importing-the-ssl-certificate)):  
![RFC Logon Security](https://github.com/meplato/store2-abap-client/blob/master/files/RFC_Logon_Security.jpg)
- You may enter additional settings according to your wishes. However, please double-check with your Meplato contact if they are supported (especially if you wish to use compression).


### Creating a service

The following services are supported:

API Service | Interface
---|---
Base Service | `/WAME/IF_MSA_BASE_SERVICE`
Catalogs Service | `/WAME/IF_MSA_CATALOGS_SERVICE`
Jobs Service | `/WAME/IF_MSA_JOBS_SERVICE`
Products Service | `/WAME/IF_MSA_PRODUCTS_SERVICE`

When creating a service, you pass on a client (see above) and you might pass on Retry Policies (we will come to that later). These two parameters are available in any service. Additionally, individual services may have further mandatory and optional parameters (the Products Service in the example below requires a catalog PIN):
```abap
DATA PRODUCTS_SERVICE TYPE REF TO /WAME/IF_MSA_PRODUCTS_SERVICE.
DATA PIN              TYPE STRING.

"...

PRODUCTS_SERVICE = /WAME/CL_MSA_FACTORY=>CREATE_PRODUCTS_SERVICE(
                       IO_CLIENT         = CLIENT "mandatory
                       IV_PIN            = PIN "mandatory for Products Service
*                       IT_RETRY_POLICIES = IT_RETRY_POLICIES "optional
                   ).
```

#### Retry Policies

You can define retry policies to let the service try several times in case an error occurs. A retry policy is a class implementing interface `/WAME/IF_MSA_RETRY_POLICY`. It defines a maximum number (count) of retries and an interval in seconds between the retries. Furthermore, each Retry Policy implements a method `IS_RELEVANT` which decides if the policy is relevant for the error.


You can set a list of retry policies for the service. In case of an error, the service will go through the list and use the first Retry Policy that declares itself relevant.

If the maximum number of retries is exceeded, the (last) error will be propagated to you program just as if it would have been if no retry policy had been set.


The ABAP client for the Meplato Store API comes with two predefined Retry Policies:
- A default retry policy: The policy is always relevant. You can set the retry count and the interval in seconds. You can get an instance by calling `/WAME/CL_MSA_FACTORY=>CREATE_REPO_DEFAULT( )`.
- A retry policy specialized on an http status. You can set a range of http status codes which defines if the policy is relevant. You can set the retry count and the interval in seconds, too, of course. You can get an instance by calling `/WAME/CL_MSA_FACTORY=>CREATE_REPO_HTTP_STATUS( )`.

Example:
```abap
DATA LO_REPO_DEFAULT TYPE REF TO /WAME/IF_MSA_REPO_DEFAULT.
DATA LO_REPO_HTTP_STATUS TYPE REF TO /WAME/IF_MSA_REPO_HTTP_STATUS.
DATA LT_STATUS_CODE_RANGE TYPE /WAME/IF_MSA_REPO_HTTP_STATUS=>TT_STATUS_CODE_RANGE.
DATA LS_STATUS_CODE_RANGE LIKE LINE OF LT_STATUS_CODE_RANGE.
DATA LT_RETRY_POLICIES TYPE /WAME/IF_MSA_RETRY_POLICY=>TT_SELF.

* Create a http status retry policy which is relevant for status 503.
* It will retry 5 times with an interval of 5 seconds.
LO_REPO_HTTP_STATUS = /WAME/CL_MSA_FACTORY=>CREATE_REPO_HTTP_STATUS( ).

LS_STATUS_CODE_RANGE-SIGN = 'I'.
LS_STATUS_CODE_RANGE-OPTION = 'EQ'.
LS_STATUS_CODE_RANGE-LOW = '503'. "Service Temporarily Unavailable
APPEND LS_STATUS_CODE_RANGE TO LT_STATUS_CODE_RANGE.

LO_REPO_HTTP_STATUS->SET_STATUS_CODE_RANGE( LT_STATUS_CODE_RANGE ).
LO_REPO_HTTP_STATUS->SET_COUNT( 5 ).
LO_REPO_HTTP_STATUS->SET_INTERVAL_IN_SECS( 5 ).

* Create a default retry policy with 5 retries after 10 seconds each
LO_REPO_DEFAULT = /WAME/CL_MSA_FACTORY=>CREATE_REPO_DEFAULT( ).
LO_REPO_DEFAULT->SET_COUNT( 5 ).
LO_REPO_DEFAULT->SET_INTERVAL_IN_SECS( 10 ).

* Prepare the list of retry policies (the http status retry policy
* will be checked first, then the default retry policy).
APPEND LO_REPO_HTTP_STATUS TO LT_RETRY_POLICIES.
APPEND LO_REPO_DEFAULT TO LT_RETRY_POLICIES.

* Set the retry policies for the service
PRODUCTS_SERVICE->SET_RETRY_POLICIES( LT_RETRY_POLICIES ).
```
In the example, the Retry Policies are set via the method `SET_RETRY_POLICIES`. This has the same effect as if they were set directly when creating the service.

It is completely optional to set Retry Policies. If they make sense may depend on the reliability of the network or other factors.

### Calling service methods

After creating the service, we would now like to call service methods like "search for catalogs" or "create a product (in a catalog)".

The data to be sent or received is stored in objects that provide a set of setter and getter methods to set or get the data. These objects thus define the payloads of the request to the service as well as of the response from the service.

Let us assume we wish to create a product in a catalog: We call the method `CREATE`of the Products Service. The data of the product (the payload) is contained in an object instance implementing interface `/WAME/IF_MSA_CREATEPRODUCT`:
```abap
DATA CREATEPRODUCT     TYPE REF TO /WAME/IF_MSA_CREATEPRODUCT.
DATA CREATEPRODUCTRESP TYPE REF TO /WAME/IF_MSA_CREATEPRODUCTRESP.
DATA EX                TYPE REF TO CX_ROOT.

* Preparing the request payload
CREATEPRODUCT->SET_SPN( `<your supplier part number` ).
CREATEPRODUCT->SET_NAME( `<your product name>` ).
CREATEPRODUCT->SET_PRICE( '10.00' ).
CREATEPRODUCT->SET_ORDERUNIT( `<your order unit (ISO code e.g. PCE)>` ).
"CREATEPRODUCT->SET_... Set further properties

TRY.
* Calling the service method
    CREATEPRODUCTRESP = PRODUCTS_SERVICE->CREATE(
                          EXPORTING
                            IO_CREATEPRODUCT = CREATEPRODUCT
                        ).

* Evaluating the response payload
    WRITE / CREATEPRODUCTRESP->GET_KIND( ).
    WRITE / CREATEPRODUCTRESP->GET_LINK( ).

* Error handling
  CATCH CX_ROOT INTO EX.
    WHILE EX IS BOUND.
      WRITE / EX->GET_TEXT( ).
      EX = EX->PREVIOUS.
    ENDWHILE.
ENDTRY.
``` 
After calling a service method, we might evaluate the response (in this case stored in an object instance implementing interface `/WAME/IF_MSA_CREATEPRODUCTRESP`) and, of course, we should implement a suitable error handling.

#### A remark on select statements

As rule, data that is to be transferred to the Meplato Store from an SAP system will somehow be selected from the SAP system’s database.

Please note that each call of a service method executes a `commit work`.

That is, service method calls must not be executed between `SELECT` and `ENDSELECT` or in any other place where the correct state of a program depends on an open database cursor.


## Documentation

Complete documentation for the Meplato Store 2 REST API can be found at
[https://developer.meplato.com/store2](https://developer.meplato.com/store2).

## Downloads

- [1.0.1.0.zip](https://github.com/meplato/store2-abap-client/blob/master/files/1.0.1.0.zip)
- [Certificate.zip](https://github.com/meplato/store2-abap-client/blob/master/files/Certificate.zip)


# License

This software is licensed under the Apache 2 license.

    Copyright (c) 2018-present WPS Management GmbH, Germany <https://www.wps-management.de>

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
