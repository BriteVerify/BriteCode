Files API
=========

*PLEASE NOTE: THIS API IS DEPRECATED AND WILL BE SUNSET IN THE NEAR FUTURE.*

The files API is a simple REST API that allows for the transfer of appropriately formatted CSV and text files of email address into the BriteVerify File Processing Platform.

NOTE: You must request File API authorization of your account otherwise the api calls will return with a "user not authorized" 401 error.

Things to Keep In Mind
----------------------
* This system is designed for files with a row size of 50,000+. It is not intended for small files and performance will be significanly impacted if many small files are sent instead of one big file.
* Batch emails into files of 50,000 or greater in size.
* You must request File API Access to be enabled on your account
* All files must be UTF-8 Encoded
* All files must have a header row with at least one column named "email"
* It is recommended that files should not exceed 500,000 records for performance reasons
* Files over 500,000 should be broken into chunks before being pushed to BriteFiles

POSTing a file
--------------
A file can be pushed into the BriteFiles platform with a simple HTTP Post.

###Parameters
* apikey: the BriteVerify apikey for an account with files authorization
* file_job[contact_file]: the file to be processed
* file_job[delimiter]: The column delimiter (comma, pipe, tab, space...)(",","|","\t"," ")
* file_job[verify_connected]: If passed as true BV Connected will be run on each email

###Example POST

```Text
curl -F apikey=my-api-key -F file_job[contact_file]=@test_emails.csv -F file_job[delimiter]=, -F file_job[verify_connected]=true -F press=OK https://files.briteverify.com/brite_files.json
```
###Response Body
```Text
{
  "id" : "a-file-job-id",
  "name" : "test.csv",
  "status" : "import_error", // "pending", "loading", "import_error", "processing", "exporting", "complete", "cancelled"
  "file_job_uri" : "https://filesapi.briteverify.com/a-file-job-id.json"
  "contact_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/contact_file&apikey=my-api-key",
  "verified_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/verified_file&apikey=my-api-key",
  "columns" : ["email", "custom_id", "fname", "lname"],
  "stats" : {"rows" : 1000, "processed" : 400, "valid" : 199, "invalid" : 201, "connected" : 78},
  "percent_complete" : 39.00,
  "email_column" : 0,
  "delimiter" : ",",
  "import_error" : "Line 1: No headers of any interest (was looking for any of: email, street, city, state, zip, first_name, last_name, full_name, phone, ip)""
}
```
File Life-cycle
---------------
Once a file has been successfully posted it will move through several states.

###Happy Path 
1. Unprocessed
2. Loading
3. Processing
4. Exporting
5. Complete

Once a file is "complete" it is ready for download. 

###Unhappy Path
If a file has issues it will move into an "error" state and notifications will be sent out. If the error occurs while the file is being loaded the status of the file will be set to "import_error" and an "import_error" message will be shown.

```Text
{
  "id" : "a-file-job-id",
  "name" : "test.csv",
  "status" : "import_error",
  "file_job_uri" : "https://filesapi.briteverify.com/a-file-job-id.json"
  "contact_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/contact_file&apikey=my-api-key",
  "verified_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/verified_file&apikey=my-api-key",
  "columns" : ["email", "custom_id", "fname", "lname"],
  "stats" : {"rows" : 1000, "processed" : 400, "valid" : 199, "invalid" : 201, "connected" : 78},
  "percent_complete" : 39.00,
  "email_column" : 0,
  "delimiter" : ",",
  "import_error" : "Line 1: No headers of any interest (was looking for any of: email, street, city, state, zip, first_name, last_name, full_name, phone, ip)""
}
```

##Status Codes
* pending
* loading
* import_error
* processing
* reprocessing
* complete


Monitoring File State & Downloading
-----------------------------------

Checking on the status of a file is easy with a simple GET request.

###Request

```Text
curl https://filesapi.briteverify.com/brite_files/my-file-job-id.json?apikey=my-api-key
```

###Response Body
```Text
{
  "id" : "a-file-job-id",
  "name" : "test.csv",
  "status" : "complete", // "pending", "loading", "error", "processing", "exporting", "complete", "cancelled"
  "file_job_uri" : "https://filesapi.briteverify.com/a-file-job-id.json"
  "contact_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/contact_file&apikey=my-api-key",
  "verified_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/verified_file&apikey=my-api-key",
  "columns" : ["email", "custom_id", "fname", "lname"],
  "stats" : {"rows" : 1000, "processed" : 400, "valid" : 299, "invalid" : 201, "connected" : 78},
  "percent_complete" : 39.00,
  "email_column" : 0,
  "delimiter" : ","
}
```

Once the status is "complete" a "verified_file_uri" will be avaialable and you can download the file from that location.

Listing File Jobs
-----------------
You can get a list of files you have pushed to BriteFiles via a simple get request.

###Request
```Text
curl https://filesapi.briteverify.com/brite_files.json?apikey=my-api-key
```

###Response
```Text
[
  {"id" : "a-file-job-id", "name" : "test.csv", "status" : "loading", "file_job_uri" : "https://filesapi.briteverify.com/a-file-job-id.json?apikey=my-api-key"},
  {"id" : "a-file-job-id", "name" : "test2.csv", "status" : "complete", "file_job_uri" : "https://filesapi.briteverify.com/a-file-job-id.json?apikey=my-api-key"}
]
```

Cancel a File
-------------

A file can be canceled by issuing a simple cancelation request via a PUT (or GET). Once a job is cancelled it will stop processing records after the cancelation request and export the file, then transition to a completed state.

###Request
```Text
curl PUT -d https://filesapi.briteverify.com/brite_files/a-file-job-id/cancel.json?apikey=my-api-key 
```

Deleting a Completed File
-------------------------
Once a file has completed processing and has been successfully downloaded.

```Text
curl -X DELETE https://filesapi.briteverify.com/brite_files/a-file-job-id.json?apikey=my-api-key
```

File Format
-----------

###Example Contact File UTF-8 Encoded

NOTE: The "customid" column is used here to denote how non-email columns can be included with the file. These columns will be ignored during processing, but will be included in the resulting file. 

```Text
email,customid
jdoe@yahoo.com,123123
james@yahoo.com, 123221312
jm@yahoo.com, 123221312
me@yahoo.com,89809089
me$yahoo.com,89809089
me@bellsouth.net,0989889
me@bellsouth.net,098922889
```

###Example Verified File UTF-8 Encoded
```Text
email,customid, email_status, disposable, role_account
jdoe@yahoo.com, 123123,email_account_invalid, FALSE, FALSE
james@yahoo.com, 123221312, valid, FALSE, FALSE
admin@briteverify.com, 123221312, valid, FALSE, TRUE
me@yaho1232o.com,89809089, email_domain_invalid, FALSE, FALSE
me$bellsouth.net,8980234239089, email_address_invalid, FALSE, FALSE
me@bellsouth.net,0989889, unknown, FALSE, FALSE
me@somewhere.net,098229889, accept_all, FALSE, FALSE
```

Email Statuses
--------------
* valid : inbox exists at domain
* unknown : domain and syntax are valid but inbox could not be 100% verified
* email_address_invalid : email syntax invalid
* email_domain_invalid : domain does not exists or is not email capable
* email_account_invalid : inbox does not exist at domain
* accept_all: The email domain responds valid to all emails

Disposable
----------

A temporary or "disposable" email address is one that a user has set up to live for only a short period of time for a variety of reasons. Usually you don't want to accept these types of emails, but we leave that up to how you wish to implement your own applications. These emails are just like any other emails and the "status" will reflect that. However, the disposable flag will be present if the email is from a known temporary email provider.

Role Addresses
--------------

Role address are email address that are set aside for functions, not people. They’re often forwarded to a group or department inside a company, and they can change owners frequently.  Sending to a role address can quickly lead to spam complaints. Also, unused role accounts are often converted to spam traps, which will also get you into a ton of trouble. Either way, it’s a darn good idea to toss these aside. However, technically speaking, they can be "deliverable" addresses. So BriteVerify does not mark them as invalid. Instead we have a role_address flag to let you know to sending to the address is not advisable, but ultimately up to you. 

Some examples of role addresses are postmaster, sales, admin, info, webmaster, etc. One role address in particular that is extremely risky is the postmaster address. Sending an email to that one is like driving drunk to a police station... not that brite ;-)






