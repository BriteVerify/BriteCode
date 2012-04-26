Files API
=========
The files API is a simple REST API that allows for the transfer of appropriately formatted CSV and text files of email address into the BriteVerify File Processing Platform.

Things to Keep In Mind
----------------------
* All files must be UTF-8 Encoded
* All files must have a header row with at least one column named "email"
* It is recommended that files should no exceed 500,000 records for performance reasons
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
curl POST -F apikey=my-api-key -F file_job[contact_file]=@test.csv -F file_job[delimiter]=, -F file_job[verify_connected]=true -F press=OK https://filesapi.briteverify.com/brite_files.json
```
###Response Body
```Text
{
  "id" : "a-file-job-id",
  "name" : "test.csv",
  "status" : "error", // "pending", "loading", "error", "processing", "exporting", "complete", "cancelled"
  "file_job_uri" : "https://filesapi.briteverify.com/a-file-job-id.json"
  "contact_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/contact_file&apikey=my-api-key",
  "verified_file_uri" : "https://filesapi.briteverify.com/a-file-job-id/verified_file&apikey=my-api-key",
  "columns" : ["email", "custom_id", "fname", "lname"],
  "stats" : {"rows" : 1000, "processed" : 400, "valid" : 199, "invalid" : 201, "connected" : 78},
  "percent_complete" : 39.00,
  "email_column" : 0,
  "delimiter" : ",",
  "errors" : ["contact_file", "line 483 contains illegal character"]
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
If a file has issues it will move into an "error" state and notifications will be sent out. 

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
email,customid,email_status,connected
jdoe@yahoo.com,123123, email_account_invalid, null
james@yahoo.com, 123221312, valid, true
jm@yahoo.com, 123221312, valid, false
me@yaho1232o.com,89809089, email_domain_invalid, null
me$bellsouth.net,8980234239089, email_address_invalid, null
me@bellsouth.net,0989889, unknown, true
me@somewhere.net,098229889, accept_all, true
```

Email Statuses
--------------
* valid : inbox exists at domain
* unknown : domain and syntax are valid but inbox could not be 100% verified
* email_address_invalid : email syntax invalid
* email_domain_invalid : domain does not exists or is not email capable
* email_account_invalid : inbox does not exist at domain
* connected : email is valid and connected to one or more active online networks
* accept_all: The email domain responds valid to all emails



