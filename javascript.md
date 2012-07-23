BriteVerify JavaScript API
==========================

NOTE: This API is deprecated and will be replaced in the near future. The documentation below is intended for users who have already implemented the JS API.


The BriteVerify JavaScript API sits atop jQuery and the functions follow a similar pattern to traditional jQuery AJAX calls in that an optional callback function may be passed to that will execute when the call completes. All BriteVerify functions are contained within the briteVerify namespace to avoid potential naming conflicts. All the functions follow a similar pattern for their method signature: value, callback. The first parameter is the value to be verified, either a simple string for functions such as email, or a JSON object, as with address. The second parameter it the callback function to be executed when the verification is complete. This function will always have the resulting object passed into it. This is illustrated in the following code example:

```JavaScript

jQuery.briteVerify.email("myemail@somedomain.com", function(email){
	alert(email.address); // myemail@somedomain.com
	alert(email.status); // valid
	alert(email.errors); // null
} 
```

To get started just include the BriteVerify JavaScript library.

```HTML
<script type="text/javascript" src="https://api.briteverify.com/users/your-username/api.js?version=01"></script>
```

You can find your include URL by logging into BriteVerify and clicking on the "Account" link.

![alt text](https://github.com/BriteVerify/BriteCode/raw/master/images/bv_account_js.jpg)

Adding Trusted Domains
----------------------

The JavaScript API uses domain based authentication to ensure that your scripts only run on authorized websites. In order for the JavaScript API to work in production mode you have to login to BriteVerify and add your trusted domains.

1. Click on the "Account" Link
2. Click on the "Edit" Link
3. Scroll to the "Trusted Domains" section
4. Type in a trusted domain, i.e., www.mywebsite.com
5. Click Add
6. Repeat for as many domains as needed


Handling Errors
---------------

###Errors Array

If the value cannot be verified, the response object will contain an array of errors. The errors array is an array of arrays. Each individual error is an array where the first value is the name of the attribute on which the error occurred and the second is the error message itself. Almost always there is only one error per field. So checking for additional errors is usually pointless. However, it can be valuable to know to which part of the value the error applies. For example, with the email response object contains the address, domain, and account attributes and the error will apply to one of those.

```JavaScript

jQuery.briteVerify.email("myemail@somedomain.com", function(email){
  // first check to see if there are any errors
  if(email.errors){
  	// get the first error in the array
  	var error = email.errors[0];
  	var attr = error[0];
  	var msg = error[1];
  	// so a different error message base on the attr
  	if(attr == "address"){
  		alert("incorrect email syntax");
  	}else if(attr == "account"){
  		alert("the account was not found on the domain");
  	}else if(attr == "domain"){
  		alert("the is not valid");
  	}
  }
} 
```

###Errors JSON Object

In addition to the basic errors array, there is also an errors_json object that is essential a utility object for the errors array. It makes coding a little easier and cleaner.

```JavaScript
jQuery.briteVerify.email("myemail@somedomain.com", function(email){
	// first check to see if there are any errors
	if(email.errors_json){
		// so a different error message base on the attr
		if(email.errors_json.address){
			alert("incorrect email syntax");
		}else if(email.errors_json.account){
			alert("the account was not found on the domain");
		}else if(email.errors_json.domain){
			alert("the is not valid");
		}
  }
} 
```

###User Account Errors

Should your account surpass its daily test transaction limit for unauthorized domains, or if your account balance reaches 0, an error will be returned on the "user" attribute. It is a best practice to ignore these messages, but they can be helpful for testing purposes. A notification will be sent out prior to any user account hitting a zero balance, or if auto-billing is enabled a billing notification will be sent.

```JavaScript
jQuery.briteVerify.email("myemail@somedomain.com", function(email){
	// first check to see if there are any errors
	if(email.errors_json){
		// so a different error message base on the attr
		if(email.errors_json.user){
			alert("user account: " + email.errors_json.user); //"overbalance"
		}
  }
}
``` 
