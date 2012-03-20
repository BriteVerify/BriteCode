Contact API
===========

The contact API let's you verify an entire business card of contact information in one simple request. So instead of having to 4 requests to verify a contact's name, email, phone, and address, you just make one call, easy peasy lemon squeezy. Heck, we even through an IP address verification in there for good measure.

So using the Contact API is just like using the other BriteVerify APIs except instead of passing a single data element to be verified, you are passing all the contact info associated with a person. The only big difference technically is how you format the parameters of the GET request. When using the Contact API you wrap each parameter in a contact, e.g., contact[data_element].

Here is an example of a GET request that will just verify a contact's email:

https://bpi.briteverify.com/contacts.json?contact[email]=james@example.com&apikey=your-api-key

This bracketed technique is a very common for passing objects as parameters instead of single values. Anyone familiar with Ruby on Rails or with how jQuery serializes JSON objects will recognize this pattern.

Now let's get into some examples.

Full Contact Verification
-------------------------

In this example we are going to just pass all our contact info in a single request. For purposes of getting started, let's just assume all this data is valid. If you have not read up on the other APIs, it might be helpful to look them over since Contact is a composite API that assembles them all together. 

```html

https://bpi.briteverify.com/contacts.json?contact[name]=James+McLachlan&contact[email]=james@example.com&contact[phone]=7045251234&contact[ip]=174.96.214.3&contact[street]=325+Example+Pl&contact[zip]=101223&apikey=your-api-key
```

```JavaScript
{
  "name":{
    "fullname":"James McLachlan",
    "first":"James",
    "last":"McLachlan",
    "gender":"male",
    "salutation":"Mr. McLachlan",
    "status":"valid"
  },
  "email":{
    "address":"james@example.com",
    "account":"james",
    "domain":"example.com",
    "status":"valid",
    "connected":true
  },
  "phone":{
    "number":"7045251234",
    "areacode":"704",
    "prefix":"525",
    "suffix":"5526",
    "latitude":"35.175301",
    "longitude":"-80.877701",
    "city":"Charlotte",
    "state":"NC",
    "country":"United States of America",
    "country_code":"US",
    "timezone":"Eastern Time",
    "timezone_code":"05",
    "status":"valid"
  },
  "ip":{
    "address":"174.96.214.3",
    "isp":"Road Runner Holdco LLC",
    "domain":"rr.com",
    "zip":"28201",
    "city":"Charlotte",
    "region":"North Carolina",
    "country":"United States",
    "country_code":"US",
    "longitude":"-80.843127",
    "latitude":"35.227087",
    "status":"valid"
  },
  "address":{
    "street":"325 Example Pl",
    "city":"Charlotte",
    "state":"North Carolina",
    "state_code":"NC",
    "zip":"10123",
    "plus4":"2803",
    "country":"United States of America",
    "country_code":"US",
    "congressional_district":"09",
    "address_type":"Street",
    "status":"valid","corrected":true
  },
  "duration":0.745678305
}
```

That is a big response, but hopefully it is pretty straight forward.

Errors & Error Codes
--------------------

One nice thing about the Contact API is that it makes it really easy to know when something is wrong with some of the contact data. If any of the data elements are invalid an errors hash and an error codes hash is added to the response. This way you don't have to search through the entire response body if you are just trying to find out which data elements are invalid.

So let's look at a simple example with email and phone.

https://bpi.briteverify.com/contacts.json?contact[email]=james@nowhere.com&contact[phone]=7041231234&apikey=your-api-key

```JavaScript
{
  "errors":{"email":"Email domain invalid", "phone":"Invalid prefix"},
  "error_codes":{"email":"email_domain_invalid","phone":"invalid_prefix"},
  "email":{
    "address":"james@nowhere.com",
    "account":"james",
    "domain":"nowhere.com",
    "status":"invalid",
    "error_code":"email_domain_invalid",
    "error":"Email domain invalid"
  },
  "phone":{
    "number":"7041231234",
    "areacode":"704",
    "prefix":"123",
    "suffix":"1234",
    "state":"NC",
    "country":"United States of America",
    "country_code":"US",
    "status":"invalid",
    "error":"Invalid prefix",
    "error_code":"invalid_prefix"
  },
  "duration":1.446247426
}
```

So what we care about are the first two arrays, errors and error_codes. 

Errors is basically a humanized version of the error codes, which represent what actually is invalid about the given data element. Refer to the individual APIs documentation for more information about which error codes can be returned for a given element.

There we go. That is the Contact API. We will be adding more documentation, examples, and How-Tos soon. In the meantime, have fun.

 