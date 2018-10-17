# REST API Guideline
This article scope is to help designing a strong Rest API

## Useful resources
## Links
* [A guide to RESTful API design: 35+ must-reads](https://techbeacon.com/guide-restful-api-design-35-must-reads#.WcVeaYqsgc0.twitter)

* [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md#51-errors)

* [Spring Boot REST API Projects With Code Examples](https://dzone.com/articles/spring-boot-rest-api-projects-with-code-examples?utm_source=Top%205&utm_medium=email&utm_campaign=Top%205%202018-01-193)

### Books

* [REST_API_Design_Rulebook](books/REST_API_Design_Rulebook.pdf)
* [RESTful_Web_APIs](books/RESTful_Web_APIs.pdf)

## Intro
This REST API Guidelines should help developers to build a consistent and easy to use api.

## Taxonomy
### Errors
Errors are defined as a client passing invalid data to the service and the service correctly rejecting that data. Examples include invalid credentials, incorrect parameters, unknown version IDs, or similar. These are generally "4xx" HTTP error codes and are the result of a client passing incorrect or invalid data.

### Faults
Faults are defined as the service failing to correctly return in response to a valid client request. These are generally "5xx" HTTP error codes.

**Note:** Calls that fail due to rate limiting or quota failures MUST NOT count as faults. Calls that fail as the result of a service fast-failing requests (often for its own protection) do count as faults.

### Latency
Latency is defined as how long a particular API call takes to complete, measured as closely to the client as possible. This metric applies to both synchronous and asynchronous APIs in the same way. For long running calls, the latency is measured on the initial request and measures how long that call (not the overall operation) takes to complete.

### Time to complete
Services that expose long operations MUST track "Time to Complete" metrics around those operations.

### Long running API faults
Long Running faults MUST roll up as Faults into the overall Availability metrics.

## Client guidance
To ensure the best possible experience for clients talking to a REST service, clients SHOULD adhere to the following best practices:

### Ignore rule
For loosely coupled clients where the exact shape of the data is not known before the call, if the server returns something the client wasn't expecting, the client MUST safely ignore it.

Some services MAY add fields to responses without changing versions numbers. Services that do so MUST make this clear in their documentation and clients MUST ignore unknown fields.

### Variable order rule
Clients MUST NOT rely on the order in which data appears in JSON service responses.


## Consistency fundamentals

### URL structure
Humans SHOULD be able to easily read and construct URLs.

This facilitates discovery and eases adoption on platforms without a well-supported client library.

An example of a well-structured URL is: `https://api.contoso.com/v1.0/people/jdoe@contoso.com/inbox`

### URL length
HTTP does not place a predefined limit on the length of a request-line. [...] A server that receives a request-target longer than any URI it wishes to parse MUST respond with a 414 (URI Too Long) status code.

### Canonical identifier
In addition to friendly URLs, resources that can be moved or be renamed SHOULD expose a URL that contains a unique stable identifier. It MAY be necessary to interact with the service to obtain a stable URL from the friendly name for the resource.

The stable identifier is not required to be a GUID.

An example of a URL containing a canonical identifier is: `https://api.contoso.com/v1.0/people/7011042402/inbox`

### Supported methods
Operations MUST use the proper HTTP methods whenever possible, and operation idempotency MUST be respected. HTTP methods are frequently referred to as the HTTP verbs. The terms are synonymous in this context, however the HTTP specification uses the term method.

Below is a list of methods that your service should support

**Method** |	**Description**															| Is Idempotent | Is Safe
---------- | --------------------------------------------------------------------------	| -------------	| ---------
  GET	   |  Return the current value of an object 									|True			|True
  PUT 	   |  Replace an object, or create a named object, when applicable				|True			|False
  DELETE   |  Delete an object 															|True			|False
  POST	   |  Create a new object based on the data provided, or submit a command 		|False 			|False
  HEAD 	   |  Return metadata of an object for a GET response. Resources that support the GET method MAY support the HEAD method as well | True			|True
  PATCH	   |  Apply a partial update to an object 										|False 			|False
  OPTIONS  |  Get information about a request; see below for details.					|True 			|True


#### When should we use PUT and when should we use POST?
Use PUT when you can update a resource completely through a specific resource. For instance, if you know that an article resides at http://example.org/article/1234, you can PUT a new resource representation of this article directly through a PUT on this URL.

If you do not know the actual resource location, for instance, when you add a new article, but do not have any idea where to store it, you can POST it to an URL, and let the server decide the actual URL.

#### When should we use the PATCH HTTP method?
The HTTP methods PATCH can be used to update partial resources. For instance, when you only need to update one field of the resource, PUTting a complete resource representation might be cumbersome and utilizes more bandwidth

#### What are idempotent and/or safe methods?
Safe methods are HTTP methods that do not modify resources. For instance, using GET or HEAD on a resource URL, should NEVER change the resource.

### Standard request header

Header 		  		|Type 							  | Is Description
------------------ 	| -----------------------------   | -------------
Date	   		  	|Date					  		  |Timestamp of the request, based on the client's clock, in RFC 5322 date and time format. The server SHOULD NOT make any assumptions about the accuracy of the client's clock. This header MAY be included in the request, but MUST be in this format when supplied. Greenwich Mean Time (GMT) MUST be in this format when supplied. Greenwich Mean Time (GMT) MUST be used as the time zone reference for this header when it is provided. For example: Wed, 24 Aug 2016 18:41:30 GMT. Note that GMT is exactly equal to UTC (Coordinated Universal Time) for this purpose.
Accept   		  	|Content type			  		  |The requested content type for the response such as: <br> * application/xml <br> * test/xml * application/json <br> * text/javascript
Accept-Encodinf 	|Gzip, defalte			  		  |REST endpoints SHOULD support GZIP and DEFLATE encoding, when applicable. For very large resources, services MAY ignore and return uncompressed data.
Accept-Language 	|"en", "es", etc.		  		  |Specifies the preferred language for the response. Services are not required to support this, but if a service supports localization it MUST do so through the Accept-Language header.
Accept-Charset  	|Charset type like "UTF-8" 		  |Default is UTF-8, but services SHOULD be able to handle ISO-8859-1.
Content-Type	  	|Content type			  		  |Mime type of request body (PUT/POST/PATCH)
Prefer  		  	|return=minimal, return=representation |If the return=minimal preference is specified, services SHOULD return an empty body in response to a successful insert or update. If return=representation is specified, services SHOULD return the created or updated resource in the response. Services SHOULD support this header if they have scenarios where clients would sometimes benefit from responses, but sometimes the response would impose too much of a hit on bandwidth.
If-Match, <br> If-None-Match, <br>If-Range |String			  |Services that support updates to resources using optimistic concurrency control MUST support the If-Match header to do so. Services MAY also use other headers related to ETags as long as they follow the HTTP specification.

### Standard response header
Services SHOULD return the following response headers, except where noted in the "required" column.

Header 		   		|Type		  					 | Is Description
------------------ 	| ----------------------------   | -------------
Date	   		  	|All responses			  		 |Timestamp the response was processed, based on the server's clock, in RFC 5322 date and time format. This header MUST be included in the response. Greenwich Mean Time (GMT) MUST be used as the time zone reference for this header. For example: Wed, 24 Aug 2016 18:41:30 GMT. Note that GMT is exactly equal to UTC(Coordinated Universal	Time) for this purpose.
Content-Type	  	|All responses			  		 |The content type
Content-Encoding 	|All responses		  	  		 |GZIP or DEFLATE, as appropriate
Preference-Applied	|When specified in request  	 |Whether a preference indicated in the Prefer request header 
ETag 				|When the requested resource has an enity tag |The ETag response-header field provides the current value of the entity tag for the requested variant. Used with If-Match, If-None-Match and If-Range to implement optimistic concurrency control.

## CORS
[CORS (Cross Origin Resource Sharing)](https://www.w3.org/TR/access-control/)

## Versioning
**API MUST support explicit versioning.** It's critical that clients can count on services to be stable over time, and it's critical that services can add features and make changes.

#### When to version
Services MUST increment their version number in response to any breaking API change. See the following section for a detailed discussion of what constitutes a breaking change. Services MAY increment their version number for nonbreaking changes as well, if desired.

Use a new major version number to signal that support for existing clients will be deprecated in the future. When introducing a new major version, services MUST provide a clear upgrade path for existing clients and develop a plan for deprecation that is consistent with their business group's policies. Services SHOULD use a new minor version number for all other changes.

Online documentation of versioned services MUST indicate the current support status of each previous API version and provide a path to the latest version.


## HTTP Status Codes
#### 2xx Success
* 200 OK
* 201 Created
* 204 No Content

#### 4xx Client Error
* 400 Bad Request
* 401 Unauthorized
* 403 Forbidden
* 404 Not Found
* 409 Conflict

#### 5xx Server Error
* 500 Internal Server Error

## Naming guidelines
### Approach
Naming policies should aid developers in discovering functionality without having to constantly refer to documentation. Use of common patterns and standard conventions greatly aids developers in correctly guessing common property names and meanings. Services SHOULD use verbose naming patterns and SHOULD NOT use abbreviations other than acronyms that are the dominant mode of expression in the domain being represented by the API, (e.g. Url).

### Resource Naming Best Practices
#### Use nouns to represent resources
RESTful URI should refer to a resource that is a thing (noun) instead of referring to an action (verb) because nouns have properties as verbs do not – similar to resources have attributes. Some example of resource are:
* Users of the system
* User Accounts
* Network Devices

and their resource URIs can be designed as below:
```
http://api.example.com/device-management/managed-devices
http://api.example.com/device-management/managed-devices/{device-id}
http://api.example.com/user-management/users/
http://api.example.com/user-management/users/{id}
```

For more clarity, let’s divide the resource archetypes into four categories (document, collection, store and controller) and then you should always target to put a resource into one archetype and then use it’s naming convention consistently:

1. **document**

	A document resource is a singular concept that is akin to an object instance or database record. In REST, you can view it as a single resource inside resource collection. A document’s state representation typically includes both fields with values and links to other related resources.

	Use “singular” name to denote document resource archetype.
	```
	http://api.example.com/device-management/managed-devices/{device-id}
	http://api.example.com/user-management/users/{id}
	http://api.example.com/user-management/users/admin
	```

2. **collection**

	A collection resource is a server-managed directory of resources. Clients may propose new resources to be added to a collection. However, it is up to the collection to choose to create a new resource, or not. A collection resource chooses what it wants to contain and also decides the URIs of each contained resource.

	Use “plural” name to denote collection resource archetype.
	```
	http://api.example.com/device-management/managed-devices
	http://api.example.com/user-management/users
	http://api.example.com/user-management/users/{id}/accounts
	```

3. **store**

	A store is a client-managed resource repository. A store resource lets an API client put resources in, get them back out, and decide when to delete them. A store never generates new URIs. Instead, each stored resource has a URI that was chosen by a client when it was initially put into the store.

	Use “plural” name to denote store resource archetype.

	```
	http://api.example.com/cart-management/users/{id}/carts
	http://api.example.com/song-management/users/{id}/playlists
	```

4. **controller**

	A controller resource models a procedural concept. Controller resources are like executable functions, with parameters and return values; inputs and outputs.

	Use “verb” to denote controller archetype.

	```
	http://api.example.com/cart-management/users/{id}/cart/checkout
	http://api.example.com/song-management/users/{id}/playlist/play
	```

#### Be Consistent
Use consistent resource naming conventions and URI formatting for minimum ambiguily and maximum readability and maintainability. You may implement below design hints to achieve consistency:

1. **Use forward slash (/) to indice a hierarchical relationships**
	The forward slash (/) character is used in the path portion of the URI to indicate a hierarchical relationship between resources.

	```
	http://api.example.com/device-management
	http://api.example.com/device-management/managed-devices
	http://api.example.com/device-management/managed-devices/{id}
	http://api.example.com/device-management/managed-devices/{id}/scripts
	http://api.example.com/device-management/managed-devices/{id}/scripts/{id}
	```

2. **Do not use trailing forward slash (/) in URIs**
	As the last character within a URI’s path, a forward slash (/) adds no semantic value and may cause confusion. It’s better to drop them completely.
3. **Use hyphens (-) to improve the readability of URIs**
	To make your URIs easy for people to scan and interpret, use the hyphen (-) character to improve the readability of names in long path segments.
4. **Do not use underscores ( _ )**
5. **Use lowercase letters in URIs**
6. **Do not use file extenstions**





















