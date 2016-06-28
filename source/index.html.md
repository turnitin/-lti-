---
title: API Reference

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/tripit/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

```
API Endpoints

Sandbox: https://sandbox.turnitin.com/api/lti/1p0/assignment
Global: https://api.turnitin.com/api/lti/1p0/assignment
UK: https://api.turnitinuk.com/api/lti/1p0/assignment
```

The core integration at Turnitin is built around the [LTI 1.1 standard](https://www.imsglobal.org/specs/ltiv1p1p1/implementation-guide).
Our API follows the specification closely, but also adopts **extensions** which
can be used to deeping the integration.

Your account key and secret can be configured in your dashboard by logging into Turnitin.

## What is LTI?

```html
<form action="https://api.turnitin.com/api/lti/1p0/assignment" method="post">
  <input type="hidden" name="context_id" value="456434513"/>
  ...
  <input type="submit" value="Press to continue to external tool"/>
</form>
```

LTI is a specification that defines a way for a `Tool Consumer` (usually an LMS)
to communicate with a `Tool Provider` (Turnitin, for example). In a way
that is standardised, extensible, and secure.

This takes shape with an HTML form request, think of it as a login for a user
or single sign on into a `Tool Provider`. The idea being the user can gracefully
enter an out-of-bound tool without having to login to another system. It's simply
a browser-to-server form request.

Some `Tool Consumers` take it a step further and do this in an iframe to give the
student an even more seamless integration.

Good LTI 1.1 Examples (*NOTE: Canvas automatically submit the form using JS to make the whole LTI process seamless*):

* [Student view](https://vimeo.com/155202201)
* [Instructor view](https://vimeo.com/155202655)

## Why LTI?
LTI allows for a better integration than other routes offered. The main benefit of LTI
is the fact that Turnitin can constantly push updates to both instructors and users
without having to worry about breaking integrations, without having to ask partners
to change API calls.

# Basic Turnitin LTI Launch
> An example Turnitin LTI launch form. (LTI is just a form request)

```html
<form action="https://api.turnitin.com/api/lti/1p0/assignment" method="post">
  <!-- A context is a class in Turnitin -->
  <input type="hidden" name="context_id" value="456434513"/>
  <input type="hidden" name="context_title" value="Design of Personal Environments"/>

  <!-- A resource is an assignment in Turnitin -->
  <input type="hidden" name="resource_link_description" value="A weekly blog."/>
  <input type="hidden" name="resource_link_id" value="120988f929-274612"/>
  <input type="hidden" name="resource_link_title" value="Weekly Blog"/>

  <!-- These parameters represent the user that's launching this form -->
  <input type="hidden" name="lis_person_contact_email_primary" value="user@school.edu"/>
  <input type="hidden" name="lis_person_name_family" value="Public"/>
  <input type="hidden" name="lis_person_name_full" value="Jane Q. Public"/>
  <input type="hidden" name="lis_person_name_given" value="Given"/>
  <input type="hidden" name="roles" value="Instructor"/>
  <input type="hidden" name="user_id" value="292832126"/>

  <!-- These represent the actual submission and an endpoint to send the grade back to -->
  <input type="hidden" name="lis_outcome_service_url" value="https://example.com/outcomes"/>
  <input type="hidden" name="lis_result_sourcedid" value="feb-123-456-2929::28883"/>

  <!-- How the request is secured -->
  <input type="hidden" name="oauth_consumer_key" value="12345"/>
  <input type="hidden" name="oauth_nonce" value="93ac608e18a7d41dec8f7219e1bf6a17"/>
  <input type="hidden" name="oauth_signature" value="QWgJfKpJNDrpncgO9oXxJb8vHiE="/>
  <input type="hidden" name="oauth_signature_method" value="HMAC-SHA1"/>
  <input type="hidden" name="oauth_timestamp" value="1348093590"/>
  <input type="hidden" name="oauth_version" value="1.0"/>

  <!-- Some required parameters -->
  <input type="hidden" name="lti_message_type" value="basic-lti-launch-request"/>
  <input type="hidden" name="lti_version" value="LTI-1p0"/>
  <input type="hidden" name="launch_presentation_locale" value="en-US"/>

  <!-- Change the CSS of the Tool Provider to suite the LMS theme -->
  <input type="hidden" name="launch_presentation_css_url" value="https://example.com/lms.css"/>

  <input type="submit" value="Press to continue to external tool"/>
</form>
```

Although the LTI 1.1 specification defines a lot of parameters, at Turnitin we only use
a small subset of them in LTI launches. These are mostly just the **required** and **recommended**
parameters defined in the specification.

The form is generated and secured server side then output to the user like any other standard form.
The form is secured using a subset of the OAuth 1.0 specification which is used for signing `x-www-form-urlencoded`
data. More information can be found here [LTI 1.1 Signing](https://www.imsglobal.org/specs/ltiv1p1p1/implementation-guide#toc-15).

<aside class="notice">
LTI doesn't use OAuth 1.0, it just uses the signing mechanism defined in it to secure requests.
</aside>

The user - student or instructor - can then launch the form from their browser.

### Basic Parameters

Parameter | Required | Description
--------- | ------- | -----------
lti_message_type | ✓ | Used by Turnitin and is required as defined in the LTI specification. **The value should be: basic-lti-launch-request**
lti_version | ✓ | Used by Turnitin for the LTI version. **The value should be: LTI-1p0**
resource_link_id | ✓ | A UUID representing the assignment in Turnitin, which will be mapped to a Turnitin assignment ID on launch.
resource_link_title | ✗ | The assignment title, if nothing is passed we will give the instructor the ability to change it in the settings tab.
resource_link_description | ✗ | The assignment description, if nothing is passed we will give the instructor the ability to change it in the settings tab.
user_id | ✓ | A UUID representing the user launching the request.
roles | ✓ | The role of the user launching. **Can only be "Learner" or "Instructor"**.
lis_person_name_given | ✗ | First name of the person launching. Used to create the user if they don't exist. If not provided we default to using the email.
lis_person_name_family | ✗ | Last name of the person launching. Used to create the user if they don't exist. If not provided we default to using the email.
lis_person_contact_email_primary | ✓ | The email to be used when creating the user.
lis_result_sourcedid | ✗ | A UUID representing the user submission. Used in combination with `lis_outcome_service_url` to send grade callback as defined in the LTI 1.1 spec.
lis_outcome_service_url | ✗ | Used in combination with `lis_result_sourcedid` to send grade callback as defined in the LTI 1.1 spec. The URL is the endpoint Turnitin will pass the grade to.
context_id | ✓ | A UUID representing the class in Turnitin.
context_title | ✗ | Used to create/update the class title.

## Security

> To authorize, use this code:

```html
<form method="post">
  <input name="httpMethod" type="text" size="5" value="POST"/>
  <input name="URL" type="text" value="http://ip-cpod3-vm1.oak.iparadigms.com:5030/api/lti/1p0/assignment">
  <input name="oauth_version" type="text" size="4" value="1.0"/>
  <input name="oauth_consumer_key" type="text" size="64" value="61115"/>
  <input name="consumerSecret" type="text" size="64" value="abcd1234"/>
  <input name="oauth_timestamp" type="text" style="width:100px"/>
  <input type="button" value="now" onClick="freshTimestamp()" style="width:100px"/>
  <input name="oauth_nonce" type="text" style="width:100px"/>
  <input type="button" value="random" style="width:100px" onClick="freshNonce()"/>
  <input name="oauth_signature_method" type="text" size="12" value="HMAC-SHA1"/>
</form>
```

> LTI in it's simplicity is a browser form submission. It's a "passwordless" login

# Workflows

# Extensions

## Parameters

> Example form with the Turnitin Custom LTI Parameters

```html
<form action="https://api.turnitin.com/api/lti/1p0/assignment" method="post">
  <input type="hidden" name="context_id" value="456434513"/>
  ...
  <input type="hidden" name="custom_startdate" value="2014-12-10T07:43:43Z"/>
  <input type="hidden" name="custom_maxpoints" value="150"/>
  <input type="hidden" name="custom_studentpapercheck" value="1"/>
  ...
  <input type="submit" value="Press to continue to external tool"/>
</form>
```

Because LTI is just a specification and is generic enough to be used by a wide variety of `Tool Consumers` and `Tool Providers`
a lot of parameters that would make it easier to communicate between an LMS and Turnitin aren't included. Because of that
we implemented some more specific parameters that can be used to update things like the assignment start date, end date, max points (for grading).

Parameter | Description
--------- | -----------
custom_startdate | The assignment start date in ISO8601 format. **Example - 2014-12-10T07:43:43Z**
custom_duedate | The assignment due date in ISO8601 format. **Example - 2014-12-10T07:43:43Z**
custom_feedbackreleasedate | The assignment feedback release date in ISO8601 format. **Example - 2014-12-10T07:43:43Z**
custom_maxpoints | The maximum number of points available to the student (what the grade will be out of)
custom_internetcheck | Determines whether the submission is checked against internet matches
custom_studentpapercheck | Determines whether the submission is checked against student paper matches
custom_journalcheck | Determines whether the submission is checked against journal matches
custom_institutioncheck | Determines whether the submission is checked against institution matches

## Callbacks

### Outcomes
> Example Outcomes Callback

```json
{
    "lis_result_sourcedid": 12345,
    "paperid": 4321,
    "outcomes_tool_placement_url": "https://api.turnitin.com/api/lti/1p0/outcome_tool_data/4321"
}
```
For extra details on the LTI resource (the Turnitin Assignment) we have a parameter called `ext_resource_tool_placement_url`.

`ext_resource_tool_placement_url` accepts an endpoint, which we use to send a small snippet of JSON to the server.

### Resource
> Example Resource Callback

```json
{
    "resource_link_id": 12345,
    "assignmentid": 4321,
    "resource_tool_placement_url": "https://api.turnitin.com/api/lti/1p0/resource_tool_data/4321"
}
```

For extra details on the LTI resource (the Turnitin Assignment) we have a parameter called `ext_resource_tool_placement_url`.

`ext_resource_tool_placement_url` accepts an endpoint, which we use to send a small snippet of JSON to the server.

## Server-to-Server Submission
