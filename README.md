 * [Conventions](#conventions)
    * [Acknowledgments](#acknowledgments)
    * [Caution](#caution)
    * [Guiding Principles](#guiding-principles)
        * [Support a wide range of clients](#support-a-wide-range-of-clients)
        * [Be consistent](#be-consistent)
        * [Look before you leap](#look-before-you-leap)
        * [A tidy API is more important than tidy code](#a-tidy-api-is-more-important-than-tidy-code)
      * [Follow standards pragmatically](#follow-standards-pragmatically)
    * [Resources](#resources)
      * [Applies to all resources](#applies-to-all-resources)
        * [Authentication](#authentication)
        * [Versioning](#versioning)
        * [Dates](#dates)
      * [Collection Resources](#collection-resources)
      * [Design considerations for URIs](#design-considerations-for-uris)
          * [Filtering](#filtering)
          * [Filtering large collections](#filtering-large-collections)
          * [Pagination](#pagination)
          * [Time selection queries](#time-selection-queries)
          * [Bulk requests](#bulk-requests)
      * [Create resource](#create-resource)
          * [Preventing duplicate creation of resources](#preventing-duplicate-creation-of-resources)
          * [Linking/Unlinking resources together](#linkingunlinking-resources-together)
      * [Read a resource](#read-a-resource)
      * [Update a resource](#update-a-resource)
        * [Avoiding concurrent updates](#avoiding-concurrent-updates)
      * [Remove properties of a resource](#remove-properties-of-a-resource)
      * [Delete a resource](#delete-a-resource)
    * [Nesting Resources](#nesting-resources)
    * [Complex Operations (Actions)](#complex-operations-actions)
      * [The 'import' action](#the-import-action)

      
Created by [gh-md-toc](https://github.com/ekalinin/github-markdown-toc.go)


## Conventions 

Keywords have been adopted from [RFC conventions](https://tools.ietf.org/html/rfc2119) to signify requirements. 

##Acknowledgments
Current Maintainer is <joyce.stack@mendeley.com>

Original documented drafted and attributed to Matt Thomson in 2014. 

This document borrowed heavily from:

* [JSON API](http://jsonapi.org/format/) 
* [Pay Pal](https://github.com/paypal/api-standards)
* [Mark Nottingham Blog](https://www.mnot.net)


## Caution
**Don’t use RFC2616.** 

RFC2616 was the reference for HTTP, but now it’s deprecated. You’ll still see it referred to in lots of places.

Source: [mnot’s blog](https://www.mnot.net/blog/2014/06/07/rfc2616_is_dead)

Reworked into these six RFCs:

-   [RFC7230](https://tools.ietf.org/html/rfc7230) - HTTP/1.1: Message Syntax and Routing
    -   low-level message parsing and connection management
-   [RFC7231](https://tools.ietf.org/html/rfc7231) - HTTP/1.1: Semantics and Content
    -   methods, status codes and headers
-   [RFC7232](https://tools.ietf.org/html/rfc7232) - HTTP/1.1: Conditional Requests
    -   e.g., If-Modified-Since RFC7233 - HTTP/1.1: Range Requests - getting partial content
-   [RFC7234](https://tools.ietf.org/html/rfc7234) - HTTP/1.1: Caching
    -   browser and intermediary caches
-   [RFC7235](https://tools.ietf.org/html/rfc7235) - HTTP/1.1: Authentication
    -   a framework for HTTP authentication
-   [RFC5988](https://tools.ietf.org/html/rfc5988) - HTTP/1.1: Web Linking      
    https://tools.ietf.org/html/rfc5988 - Web Linking

**URI Specification**

- [URI](https://tools.ietf.org/html/rfc3986) - Uniform Resource Identifier (URI): Generic Syntax

## Guiding Principles  
 

#### Support a wide range of clients
 
* Network
	* Same Amazon region as the API 
	* Data centre with a fibre link 
	* Home internet connection 
	* Mobile network


* Device - differ in power and battery 
	* Servers
	* PCs
	* Mobile phones / tablets 
	
* Programming languages 
	* not all languages have the same support for concurrency/HTTP e.g. PHP 	 	
	
* Syncing vs non-syncing

* Release cycles
	* these vary depending on device between webapps (days) and installed apps (months) 	

#### Be consistent 

* Should look like one API, not many 

* Similar things should look similar, regardless of which service they're in or who worked on them 
	* eg if you use snake_case then it should be snake_case everywhere	 	
	
#### Look before you leap 

* APIs can be hard to change after they're released
	* some decisions can be hard to change 

#### A tidy API is more important than tidy code 

* There are more clients than API providers in an organisation so design an API that they want 
	* this can lead to difficulties in implementing features but it will be worth it   	
	

### Follow standards pragmatically 

* Use standard HTTP, and industry best practices, where appropriate:
	* avoids surprising client developers
	* tried and tested
	* better support in client-side libraries 

	
* However, there may be times where:
	* following the standard exactly would be excessively complicated
	* there is no standard
	* there are multiple competing standards, and vigorous debate about which one is right

Its more important to have a working API with actual clients than achieving perfect RESTful purity.

## Resources

### Applies to all resources

#### Authentication
coming shortly  

#### Versioning 
(this needs further clarifications)

**Golden Rule** 

		Once we've released an API, we must NOT make breaking changes to it

        (even if it's not being used, or if it's only deployed to staging, 
        or if there's only one client, or if we marked it as beta, 
        or if all of the clients are sitting in the same room as us, 
        or if we're really-really-careful-yes-we- really-mean-it-this-time)


*Content Negotiation* 

Use HTTP's content negotiation mechanism for versioning. Custom media types are used in the API to let the consumers choose the format of the data they wish to receive. This is done by adding the `Accept` header when you make a request. Media types are specific to resources, allowing them to change independently.  

	application/vnd.mendeley-<RESOURCE_TYPE>.<VERSION>+json
	
	e.g. application/vnd.mendeley-document.1+json.  The "1" here is the version number of the representation.
	
	
Both client and servers are responsible for negotiation to be successful. 

**Client Responsibilities** 	

* Clients **MUST** send an appropriate resource Media type in the header 
		* e.g. Accept: application/vnd.mendeley-document.1+json


**Server Responsibilities**   	

* Servers **MUST** respond with a Content-Type header 
		* e.g. Content-Type: application/vnd.mendeley-document.1+json 	
	

#### Dates

ISO 8601 format **MUST** be used for all dates and times e.g. 2015-02-18T04:57:56Z

**!** Unfortunately, some standard HTTP headers use their own format, defined in [RFC 2822](http://www.rfc-base.org/txt/rfc-2822.txt): Thu, 01 May 2014 10:07:28 GMT

Server-generated dates **MUST** be used everywhere as the server is the only reliable clock.


### Collection Resources

Collections of resources can be operated on using HTTP verbs. 


`GET` - Fetch a representation of the resource at the URI.

`POST` - Store the enclosed entity as a subresource of the resource at the URI.

`PUT` - Store the enclosed entity under the URI.

`DELETE` - Delete the resource at the URI.

`PATCH` - Apply partial modifications to the resource at the URI.



### Design considerations for URIs

Here is the [URI](https://tools.ietf.org/html/rfc3986) specification. 

Developers **MUST** consider how they design their URIs. They need thought and consideration. 

* any resource of significance **MUST** be given a URI 
* URIs **SHOULD** not change. 
* consider your domain when naming your resource eg. 
	*  `files` is a common resource that you may find in many domains - consider how it might be addressed 
* a URI will repeatably refer to "the same" thing  eg. 
	* `http://www.example.org/releases/1.0.tar.gz` will always only ever address the `1.0.tar.gz` file. 
	* `http://www.example.org/releases/latest.tar.gz` *MAY* also point to `1.0.tar.gz` but the intention behind it is very different - these are two resources and two concepts

* avoid including information about about ownership such as company or product eg 
	* imagine that company had a product called `BigCoolProduct` with a URI of was `http://www.fakecompany.com/bigcoolproduct/product/123`- this identifies the product BigCoolProduct which as a product resource with an id of 123. After a couple of years the BigCoolProduct gets renamed to `AwesomeCoolProduct`. We now have a situation where all URIs need to be updated because we baked in these ownership details into our URI.      

> Because implementations change, but cool URIs don’t.
>> [Roy T Fielding](http://roy.gbiv.com/untangled/2008/rest-apis-must-be-hypertext-driven) 


**Naming**

Resources **MUST** be identified using plural nouns. A single resource **MUST** be identified using a single noun. 


	* /things - identifies a collection of resources.
    * /things/(resource_identifier) - identifies a single resource within a collection


**Namespaces**

Each collection of URI templates **MUST** include a namespace at the start of the URI. This namespace identifies a collection of related resources. This avoids collisions when there is multiple resources with similar names but different functionality.  

**URI Templates**

URI templates **MUST** follow either the URI template

<code>/{namespace}/{resource}/{resource_identifier}/{sub_resource}/{sub_resource_identifier}/</code>



**HTTP Cheat Sheet**

|  | GET | POST | PUT | DELETE | PATCH | 
| ---| --- | --- | --- | --- | --- | 
| How does the HTTP spec define it? | The GET method requests transfer of a current selected representation for the target resource. | The POST method requests that the target resource process the representation enclosed in the request according to the resource’s own specific semantics. | The PUT method requests that the state of the target resource be created or replaced with the state defined by the representation enclosed in the request message payload. | The DELETE method requests that the origin server remove the association between the target resource and its current functionality. | The PATCH method requests that a set of changes described in the request entity be applied to the resource identified by the Request-URI. (RFC 5789) |
Is it safe? (no side effects) | Yes | No | No | No | No | Yes | Yes | No | Yes |
Is it idempotent? (repeating the request may give a different response, but makes no material difference to the resource) | Yes | No | Yes | Yes | No | Yes | YEs | No | Yes |
Are responses cacheable? | Yes | Only when explicitly marked as such | No | No | Only when explicitly marked as such | No | Yes | No | No |
Can requests include a body? | No | Yes | Yes | No | Yes | Yes (but no defined uses for this) | No | No | No |
What could it be used for in a RESTful API? | Fetching a representation of the resource | Creating a new resource from a representation | Replacing a resource with a representation | Deleting a resource | Applying partial updates to a resource | CORS | Fetching only the headers (e.g. pagination counts) | Nothing - breaks layered system constraint | Nothing - breaks layered system constraint   
             
             
##### Filtering 

All resources **SHOULD** provide a filtering mechanism where it is expected to return a large collections. 

Query parameters **MUST** be used to filter on resources. 

	* e.g. /dogs?type=poodle  


##### Predefined views 
In some cases you **MAY** provide a predefined server view that contains a select number of fields e.g. 

	* e.g. /dogs?view=full OR /dogs?view=small
	
The `view=full` might have all fields in the `dog` resource whereas the `view=small` may have only a small subset of fields

*Pros* 

* No runtime assembly waiting time
* Can be used in either the query parameter or in the [HTTP Prefer](http://tools.ietf.org/html/rfc7240) as outlined in [Create resource](#create-resource)

*Cons*

* Requires development time to code up the view 	

#### Selecting subset of fields
(Joyce to add )

##### Filtering large collections
Some collections are too large to return in one response e.g. catalog, institutions or location information. 

In such cases the default **SHOULD** default to returning a limited number of results or enforcing a CAP. What is the 'default' is specific to the resource in question e.g. most popular locations/institutions.

Mandatory query parameters **MAY** also be used. 
              
             
##### Pagination 

Large collection resources **MUST** have an upper bound on the number of items in the response. It is not desirable for either our servers or our clients for us to dump all the data in one go. 

If a client requires a mechanism to iterate over a collection of resources then **cursor-based** pagination **MUST** be used. 

[Link headers](https://tools.ietf.org/html/rfc5988) are used to link to other pages. Using Link headers keeps resource representations clean. 

	GET /documents
    200 OK
    Link: </documents?marker=291d3064-4f74-4932-bfc8-4277d441705b>; rel="next";
    [
    ]
   // do

<code>limit</code> **MUST** be used to indicate the upper bounded value. 

##### Time selection queries
modified_since or deleted_since or {property_name}_since **SHOULD** be provided if time selection is needed. 
            
##### Bulk requests
Developers **SHOULD NOT** write bulk APIs. 

e.g. <code>GET /dogs/id1,id2,id3,</code>         

*Why?*

* different response formats from the same URI <code>GET /dog/id1</code> should return a single resource and not a collection of resources. 
* breaks cacheability
* URL doesn’t represent a resource
* URLs have a maximum length    



------
             
### Create resource    
Creates a new resource using the POST verb. The response to a POST **MUST** be `201 Created`, with a `Location header` containing the URL where the resource can be found. The body **MUST** contain a representation of the resources (including any server-generated fields). 

*URI template*

<code>POST /{namespace}/{resource}/</code>

*Example request and response*
 	POST /datasets/drafts
    {"field": "value"}

	    201 Created
	    Location: https://api.mendeley.com/datasets/drafts/291d3064-4f74-4932-bfc8-4277d441705b
	    {"field": "value", "created": "2015-02-18T04:57:56Z"}    
    
    
Clients **MAY** use the [HTTP Prefer](http://tools.ietf.org/html/rfc7240) header to give a client a mechanism to return a full representation or not:

        Prefer: return=minimal
        Prefer: return=representation

The body of a `POST` request, and the body returned on a GET request to the URL from the `Location` header, should have the similar structures.

       POST /things {"field": "value"} 
       
       201 Created
       Location: https://api.mendeley.com/things/291d3064-4f74-4932-bfc8-4277d441705b

       GET /things/291d3064-4f74-4932-bfc8-4277d441705b 
       
       200 OK
       {"field": "value", "created": "2015-02-18T04:57:56Z"}

GET response will usually have more fields, but that’s OK. There are some some cases where this doesn’t work e.g. 

-   POST /files: body is the file bytes
-   GET /files/(id): body is the file bytes
-   extra metadata (e.g. the document that the file is attached to) goes in the headers as it’s the only place left

          
##### Preventing duplicate creation of resources

Clients **MAY** using the [*post once exactly*](https://tools.ietf.org/html/draft-nottingham-http-poe-00) pattern to avoid duplicates being created. 

First Attempt works OK: 
    
    ￼￼POST /poe/documents/291d3064-4f74-4932-bfc8-4277d441705b
    {"title": "Underwater basket weaving"}

    200 OK
    Location: /documents/7ab2c167-8e48-4fb8-85b0-73cdb8662a64

Second Attempt works errors: 

    ￼POST /poe/documents/291d3064-4f74-4932-bfc8-4277d441705b
    {"title": "Underwater basket weaving"} 

    405 Method Not Allowed
    Allow: GET
          
         
This is just a draft and has not been updated in 10+ years but is a suggested pattern.        
         
         
##### Linking/Unlinking resources together 
You **MAY** use LINK and UNLINK hypermedia links to indicate when one resource is linked/unlinked to another. 
  
*Example request*   
If you had to create a 'Dog' resource and link it to an 'Owner' then you could do:           
             
	￼￼￼POST /dogs
	Link: </owners/291d3064-4f74-4932-bfc8-4277d441705b>; rel="owner";             
             
             
*Example in pagination*
This shows the link to the 'next' item in the list. 

	GET /documents
    200 OK
    Link: </documents?marker=291d3064-4f74-4932-bfc8-4277d441705b>; rel="next";
    [
    ]
    // do             

You **MAY** use the UNLINK method to unlink one resource from another. 
             
Specified by [*this internet draft*](http://tools.ietf.org/html/draft-snell-link-method), but at the time of writing is not an approved RFC.              
             
### Read a resource       
Reads a single resource using the GET verb from a collection of resources. 

GET calls can be called multiple times without any side effects i.e. idempotent.

The response to a GET **MUST** be `200 OK`. The body **MUST** contain a representation of the resources including any server-generated fields. In the event the resource is not located then the HTTP status returned **MUST** be `404 Not Found`. 


*URI template*

<code>GET /{namespace}/{resource}/{resource_identifier}</code>

*Example request and response*

    GET /datasets/articles/{id}    
	{
		"id": "53c9523b-7535-4501-9969-93706f14672a",
		"title": "Distinct loci of lexical and semantic access deficits in aphasia: Evidence from voxel-based lesion-symptom mapping and diffusion tensor imaging",
		"doi": "10.1016/j.cortex.2015.03.004",
		"journal_id": "7be5c683-a7c2-4fe3-95c8-947b58706f2b"
	}


### Update a resource 
Updates a single resource using the PATCH verb. Applies a partial update.  

The response to a PATCH **MUST** be `200 OK`. The body **MUST** contain a representation of the resources including any updated server-generated fields. In the event the resource is not located then the HTTP status returned **MUST** be `404 Not Found`. If the patch is invalid then a `422 Unprocessable Entity` should be returned. 

If applying the PATCH puts the resources in an invalid state then a `409 Conflict` should be returned.  

Content-Type must be set to the resource's custom media type eg. application/vnd.mendeley-draft-dataset.1+json

*URI template*

<code>PATCH /{namespace}/{resource}/{resource_identifier}</code>

*Example request*

    PATCH /datasets/drafts/{id}
	{
     "name": "Testing One Two"
    }
  
Beware of cases where `PATCH /resource1` can affect the state of `/resource2`.

#### Avoiding concurrent updates
Clients **MAY** use a precondition check of `If-Unmodified-Since` header on update requests. If specified, the resource in question will not be updated if there have been any other changes since the timestamp provided. Should be specified in [RFC 2822](http://www.rfc-base.org/txt/rfc-2822.txt) format e.g. Thu, 01 May 2014 10:07:28 GMT


It **MAY** be a requirement and the server can send back `428 Precondition Required` if there is no `If-Unmodified-Since` header. A HTTP status of `412 Precondition Failed` **MUST** be returned if the update fails. 

This **MAY** be used for GET requests e.g don’t download data that hasn’t changed since the client last requested it


### Remove properties of a resource
Removes properties of a single resource using the PATCH verb and a subset (remove operation) of [JSON PATCH](https://tools.ietf.org/html/rfc6902) semantics.

Content-Type must be set to application/json-patch+json

*Example request*

    PATCH /datasets/drafts/{id}
    [
     { "op": "remove", "path": "/name" }
    ]
 
The "remove" operation removes (sets to null) the value at the target location.

The target location MUST exist for the operation to be successful.


### Delete a resource 
Deletes a single resource using the DELETE verb. The response to a DELETE **MUST** be `204 No Content`. In the event the resource is not located then the HTTP status returned **MUST** be `404 Not Found`. The action may be forbidden by a particular client so a HTTP status of `403 Forbidden` **MUST** be returned. 

*URI template*

<code>DELETE /{namespace}/{resource}/{resource_identifier}</code>

*Example request and response*

    DELETE /datasets/drafts/{id}
    
	204 no content
	
	
## Nesting Resources
Sometimes it maybe necessary to nest resources because a sub resource can't exist without a parent resource. Developers are encouraged to think about if resources need to be nested and if they could be identified by a single ID. 

URLS **MUST** not exceed two levels of nesting. 

All standards as outlined in the [Resources](#resources) section should be adhered to. 

*Problems with multiple identifiers*

Maintaining multiple IDs can make extra work for clients having to maintain the relationships between parent and child resources. 

Overhead on server developers who have to validate multiple IDs. 


*URI template*

<code>VERB /{namespace}/{resource}/{resource_identifier}/{sub_resource}/{sub_resource_identifier}</code>

*Example request*

 	GET /submissions/fb5cd024-fb53-3366-b3f2-0dd6910cb73e/attachments/60721cc1-c1f3-678c-4235-f239d6415df1
    
*Bad example*	
  
  	GET /library/documents/7135271181/file/3221525ea6f746b577b6a8ad40d89df3f41f776a/2589311
  	
In this example we have no context what the IDs are identifying.  	
  	
## Complex Operations (Actions)
Some actions don’t naturally map to a HTTP verb. Verbs such as 'activate', 'cancel', 'validate', 'accept', 'reset', 'verify', and 'deny' are examples. These are usually a procedural concept. 


In this case you **MUST** use a POST with a verb. The word `action` **MUST** be the last segment of the URI. The `action` **MUST** be used in the body of the POST. This allows us to version the content in the cases where no request body is provided e.g. resending verification email. 

The request Content Type **MUST** be `application/vnd.mendeley-{resourcetype}-action.1+json`. 

Developers are encouraged to consider resource design alternatives over using this approach. This should be used infrequently as its an anti-pattern. 

*Risks* 
(I've taken these from Pay Pals standards as I think they apply here too.)

* The URI can't be extended to include a sub-resource beyond the verb.
* There is no corresponding read actions so this can be difficult to test.
* Retrieving a history or audit of the call will have to live in another resource `e.g. send_password_reset_email_history`
 

*URI template for action on a single resource*

<code>POST /{namespace}/{resource}/{resource_id}/actions</code>  	
*Example request*  	

  	POST /datasets/drafts/bnwvgpfvhf/actions
  	
  	{
    	"action": "send_password_reset_email"
    	// any other additional information
  	}
  	
  	
### The 'import' action
Before considering making an `action` of an import action then consider: 

* What resource are you importing? 
* Should you consider reusing an existing resource with a different Content Type? 
* Look at the attributes you are using to POST to the import - are they similar to attributes in another resource? 		
* * Is their a natural fit? 
	

  	
  	
  	
