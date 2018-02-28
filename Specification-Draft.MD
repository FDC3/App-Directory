# FDC3 App Directory Specification - Draft

Directory of trusted financial applications.

## Database
Namespace: Application?

### Applications
**name (string)** - Unique identifier. Name of the applicaion. Alphanumeric & Hypens only.

title(string) - Full name of the application - for display purposes.

contact_email(string) - Publisher's contact email.

description(string) - Short decscription of the app.

screenshot(string path) - Screen Path to S3 location of screenshot of the app

date_created(date) - Date the entry was added.

active(bool) - Is the app active\valid?

signature_id(integer) - To join to Signature table and get the Signature info.

publisher_id(integer) - To join to publisher to just see who owns application

manifest_url(string) - Location where app manifest is stored

intents(string) - List of intents supported by the application

accepted_context(string) - Descriptor of Context types the application accepts

### Signature
#### The identifier used to verify an App
id(number) - Auto generated

name(string) - Name of domain/public key

type(string) - Is it a public key/domain

date_created(date) - Date api key was added

publisher_id(number) - Id of publisher who owns it

active(bool) - Is Domain active

### Publishers
#### Entitiy that owns both a Signature and an App
id(number) - auto generated

key(string) - sha256 of api key

name(string) - Name of owner of api.

date_created(date) - Date api key was added

active(boolean) - Whether the publisher is valid or not


## API
Read requests are open.  Write requests will require an API key (set per publisher). All HTTP requests should be logged for audditing purposes.

### GET /api/v1/apps
Returns a list of all applications

Example:
Request:
GET /apps
Response:
[{
  name: "appName",
  title:"My App",
  description: "excellentDescription",
  author: "authorName",
  contact_email: "email",
  screenshot: Blob,
  date_created: Date,
  manifest_url: "http://linktomanifest"
  },...
]

Future tasks
* Make the path searchable (by name, owner, etc.)
* Pagination

### POST /api/v1/apps
Add a new app.
Requirements:
```
name: Name of the application. Must be unique(cannot be other names in app directory)
domain: Name of the domain(Conditionally Optional with public key)
public_key: Public key that the user will attach(for native app. Conditional with public key)
description: Description of application. TODO: Figure out what the max length should be.
contact_email: Email of the point of contact.
screenshot: Screenshot of application. Should only be JPEGs I think. Optional,
api_key: Key to validate the post request is valid
```


### GET /api/v1/apps/:name
Returns an object of the application. Along with if this application has been validated or not.

### PATCH /api/v1/apps/:name
Updates the application object. The customizable fields are:
```
domain: Name of the domain(Conditionally Optional with public key)
public_key: Public key that the user will attach(for native app. Conditional with public key)
description: Description of application. TODO: Figure out what the max length should be.
contact_email: Email of the point of contact.
screenshot: Screenshot of application. Should only be JPEGs I think. Optional
```

### PUT /api/v1/apps/:name/activate
Activates the application object. API key is required.

### PUT /api/v1/apps/:name/deactivate
Deactivates the application object. API key is required.

### GET /api/v1/signatures
Returns a list of all signatures

Example:
```
Request:
GET /signatures
Response:
[{
  name: "signatureName",
  date_created: Date,
  publisher_id: id of publisher that owns this
  },...
]
```

### POST /api/v1/signatures
Add a signature

Example:
```
Request:
POST /signatures
{
  name: "name of signature",
  publisher_id: "id of publisher this belongs to"
}
```

### GET /api/v1/subscriptions
Returns a list of all publickeys

Example:
```
Request:
GET /publickeys
Response:
[{
  name: "publicKey",
  date_created: Date,
  publisher_id: id of publisher that owns this
  },...
]
```

## View
### /upload
Upload a app through the frontend
### /
Home page where user can validate the app.

### /login
Login page

### Get project version
In order to get the version of the app-directory project, send a GET request to: `/version`.
