# Best Practices for Building REST Apis

It does not have any real guidelines or standards of API development. We have six constraints of a RESTful system: Client-server architecture, Statelessness, Cacheability, Layered system, Code on demand and a Uniform interface. But REST is just a design approach and not a framework or standard per se. Now it is easy to imagine that over the years the developers have applied multiple different approaches, and tried a variety of methods for delivering better REST API solutions. Some of them turned out not to be so great, others were adopted and are still widely used. It’s safe to say that over these 18-19 years, the software development community has worked out some key best practices that are either derived from the Roy Fielding’s dissertation or constituted through a trial and error process. Here are the 9 best practices we should consider when preparing the REST API.


## 1. Use JSON
It may seem too obvious, but REST allows using different output formats, like plain text, JSON, CSV, XML, RSS, or even HTML. For sure this may depend on the application you have and specifically on what you need your API for. If you have a public facing service which you want to be accessible through REST API almost in 99% of cases you should choose JSON as the data format used in the communication, both the payload and the response. What is more application/JSON is a generic MIME type which makes it a practical approach to use.

## 2. Use Nouns instead of Verbs
In API development REST approach can be called a resource based. So in your application, you work with resources and their collections (eg. a single book, and a list of all books). The actions on the resources are strictly defined by the HTTP methods such as GET, PUT, POST, PATCH, DELETE (and a few others, that are less relevant for this example) and only they should be used to change the state of the resource. This leads us to the endpoint URI construction. Taking the above into account properly constructed endpoint should look similar to this:

Correct way
```
GET /books/123
DELETE /books/123
POST /books
PUT /books/123
PATCH /books/123
```
-vs-

Incorrect way
```
GET /addBook123 (by the way, GET should be only used to READ data and never to change its state in any way)
GET /DeleteBooks/123
POST /DeleteAllBooks
POST /books/123/delete
```
There are certain cases where it is ok to use actions in a similar way to manipulating resources, eg.:
PUT /accounts/123/activation

Such a request create or update an activation property to a ‘123’ account, instead of simply saying the “activate” should be invoked.

It is tempting to use something like POST /activate_account/123 or PUT /account/123?activate=true but both of these are incorrect.

## 3. Name the collections using Plural Nouns
For the collections in REST API development use plural nouns. It is not just an established good practice. It actually makes it easier for humans to get an idea that something is actually a collection.

Eg. 
```
GET  /cars/123
POST /cars
GET /cars
```
-vs-
```
GET /car/123
POST /car
GET /car
```
In the first example, you can see the reference to car number 123 from the whole list of “cars” available. Using “cars” in plural form informs us that this is actually a collection of different car objects. 
In the second example, this is not so obvious, that there are other cars in the system.

## 4. Use resource nesting to show relations or hierarchy
Resource objects often have some kind of functional hierarchy or are related to each other. For example in the online store, we have ‘users’ and ‘orders’. Orders always belong to some user, therefore we may have the following endpoints structure laid out:

```
/users <- user’s list
/users/123 <- specific user
/users/123/orders <- orders list that belongs to a specific user
/users/123/orders/0001 <- specific order of a specific user
```
It is generally a good idea to limit the nesting to one level in REST API. Too many nested levels may not look too elegant. Also, it’s worth noting that a similar result may be achieved by filtering, which in our case could look like this: /orders?user=123. You will find more information on filtering in point 6.

## 5. Error Handling
Nobody likes to see the errors in a response that they’ve made to your API. But if they do, it’s good to provide them with enough information to know what has happened and possibly why. So, in API development, correct error handling should be considered as one of the best practices.

Standard HTTP error code handling is a must. There are 71 distinct HTTP status codes, so why not use them? These error codes are well defined and easily recognizable, therefore it makes perfect sense not to reinvent the wheel. 

In many cases, at least correct handling of the HTTP statuses is enough, but it is even better to provide more verbose messages and some internal code reference for even more detailed explanation. So the “perfect” error message would consist of:

HTTP Status Code
Code ID - which may be an internal reference, you may also provide a link to the API documentation containing all the code id’s
Human readable message shortly explaining the error (its cause, context or possible remedy)

Example from Twilio:
```
{
  "status": 400,
  "message": "No to number is specified",
  "code": 21201,
  "more_info": "http:\/\/www.twilio.com\/docs\/errors\/21201"
}
```

## 6. Filtering, sorting, paging, and field selection

Few of the most important features for consuming an API are filtering, sorting and paging. Resource collections are often times enormous, and when some data has to be retrieved from them, it would be simply not very efficient to always get the full list and browse it for specific items. Therefore we can use:

Filtering - to narrow down the query results by specific parameters, eg. creation date, or country
Sorting - basically allows sorting the results ascending or descending by a chosen parameter or parameters, eg. by date
Paging - uses “limit” in order to narrow down the number of results shown to a specific number, and “offset” to specify which part of the results range to be shown - this is important in cases where the number of total results is greater than the one  presented, this works like a pagination you may encounter on many websites 
Filtering, sorting, and paging are used by adding a so-called query parameter to the endpoint that is being called. These may look as follows:

Filtering:
```
GET /users?country=USA
GET /users?creation_date=2019-11-11
GET /users?creation_date=2019-11-11
```
Sorting:
```
GET /users?sort=birthdate_date:asc
GET /users?sort=birthdate_date:desc
```
Paging:
```
GET /users?limit=100
GET /users?offset=2
```
All together:
GET /users?country=USA&creation_date=2019-11-11&sort=birthdate_date:desc&limit=100&offset=2

(this query should result in the list of 100 users from the USA, which account creation date was on 11/11/2019, sorted descending by the birth date, and the presented records are on the second page, which means are from a 101-200 record number range).

While we’re at the topic of query parameters, it’s also worth mentioning “field selection”. Field selection allows you to request only part of the data that would be available for a certain object. So if the objects you are querying have a lot of data (fields) and you only need a few specific ones, you may use a query parameter to specify which ones should be included in the response. 

eg. we have a user object that has the following fields:
```
name,
surname,
username,
birthdate;
email,
telephone.
```
We want to get their username and e-mail to be used for sending a newsletter. The query may look something like this:
```
GET /users/123?fields=username,email (for one specific user)
GET /users?fields=username,email (for a full list of users)
```

## 7. Versioning
Versioning your REST API is a good approach to take right from the start. This will allow you to introduce changes to the data structure or specific actions, even if they are breaking/non-backward compatible changes. At some point, you might end up managing more than one API versions. But this will allow you to introduce modifications and improve services on one hand, and on another not to lose a part of your API’s users as some of them might be either reluctant to change (their integrations) or are just slow adopters that need time in order to introduce changes on their side. 

How to approach versioning? There are generally two routes to take. Implement the version in the request header or in the endpoint URI. The latter one seems to be more often used, and actually, to some extent, facilitates the readability and discoverability. Although some services (such as Stripe) combine both methods, this might actually be an “overkill” for your service. 

Examples of endpoint URI versioning:
```
https://us6.api.mailchimp.com/3.0/ (major + minor version indication)
https://api.stripe.com/v1/ (major version indication only)
https://developer.github.com/v3/  (major version indication only)
https://api.twilio.com/2010-04-01/ (date based indication)
```
## 8. API Documentation
It is extremely important to have API documentation published. Not only developers like to know what they are dealing with when making an integration, but this also allows potential users to see what is made available via your API. 

API documentation should provide information about the available endpoints and methods, preferably with request/response examples, possible response codes, information about the authorization, available limits or throttling. Publishing the documentation in the form of a browsable web page with interactive options, curl examples, maybe even a playground is a good idea. 

Remember that the API documentation also shows how much your company cares. Well written, complete and nicely presented docs will be appreciated by the developers and/or partners and may serve as examples of how should it be made. Whereas sloppy, incomplete documentation that lacks examples, contains errors or is not up to date may be harmful to your company image. 

Good API documentation examples:
```
Mailchimp
Twilio
Stripe
```
## 9. Using SSL/TLS
Always use SSL/TLS to encrypt the communication with your API.



