---
layout: doc-article
permalink: /webapi/b2c/quickstart/enroll-users/
section: webapi
subsection: b2c
categorie: Quickstart
title: Enroll End-Users
description: "End Users should give access to their data (enrollment)if they want to use a developer app."
redirect_from: 
    - /webapi/b2c/quickstart/
require: api-reference
---

<div class="notification notification-empty-bg">
{% capture short %}
**Requirement**:
  - 👤 A [registered Application]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#article) in Stellantis systems and [its credentials]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#3%EF%B8%8F⃣-receive-your-credentials).
  - 🔑 Your Application [private key]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#2%EF%B8%8F⃣-generates-encryption-keys--csr).

**Tutorial Output**:
  - 🪙 An Access Token allowing requesting data about a user enrolled in your App.
{% endcapture %}
{{ short | markdownify}}
</div>

This tutorial explains how to enroll an End User in order to obtain an **Access Token** using the **OAuth2** Authorization Code Flow. This token allows **requesting data** about the user.

First, we will need to verify if the [vehicle is eligible](#1️⃣-vehicle-capabilities) for your use case. Then, we will see how to implement the **OAuth2 Authorization Code Flow** by creating the  [consent request](#2️⃣-request-consent) in your App and finally, to exchange the Authorization Code against the [access token](#3️⃣-access-token).

{% include realms.md %}

## 1️⃣ Vehicle Capabilities

First of all, before enrolling an end user, you must make sure that the targeted vehicle is eligible to your application use case.

Indeed, as explained in [Data Availability & Scopes]({{site.baseurl}}/webapi/b2c/quickstart/data-availability-scopes/#article), all vehicles are not able to generate all the data listed in [API Reference]({{site.baseurl}}/webapi/b2c/api-reference/references/#article). When data is not generated by a vehicle, the field or object is not outputted in the API response.

That's why you need to verify that a vehicle is able to provide the data required for your application use case. It can be done using the **capability API**.

<img src="{{site.baseurl}}/assets/images/b2c-vhcl-capabilities.svg" alt="b2c-vhcl-capabilities" style="width: 550px">

The capability API will return the **technical capabilities** of the vehicles in terms of datasets. These capability datasets are subsets of [vehicles scopes]({{site.baseurl}}/webapi/b2c/quickstart/data-availability-scopes/#list-of-scopes). Relations between APIs endpoints, data & scopes can be found in the [Data Catalog]({{site.baseurl}}/connected-vehicles/data-catalog/#article) (each data has a *"B2C Scopes"* section) and the [API Reference]({{site.baseurl}}/webapi/b2c/api-reference/references/#article) (APIs endpoints have an *"authorization>scope"* dropdown).

Using the response of the capability API, and knowing the required data for your use case, you can determine whether the vehicle is **eligible to your service or not**. If the vehicle is not eligible, you must not enroll the user under this use case, but you can find a fallback use case fitting the vehicle capabilities.

For example if your use case requires the vehicle location, but the scope `data:position` is not returned in the capability API response for this vehicle, then you should probably not offer your service to this user.

> **Technical capability & Vehicle eligibility:** a data being available within the capability API doesn't mean that the user is able to subscribe third party services, checkout [vehicle services]({{site.baseurl}}/webapi/b2c/quickstart/data-availability-scopes/#-vehicle-data-upload).


#### Capability API Example

Type|Name|Value|Description|Required
-|-|-|-|-
Path Parameter|`{vin}`|`{vin}`| Vehicle VIN to check. | Yes
Query Parameter|`client_id`|`<client_id>`| *Client Id* of your application.|Yes
Query Parameter|`extension`|`onboardCapabilities`| Filters only capability data of this API. | No
Header|`x-introspect-realm`|`<realm>`| Vehicle manufacturer [realm](#manufacturers-brands--realms). | Yes
File|Client Certificate|`path/to/client_certificate.pem`| Your SSL certificate for authentication in Stelantis network.|Yes
File|Private Key|`path/to/key.pem`| Your Private Key file.| Yes
File|CA Certificate|`path/to/ca_certificate.pem`| Stellantis CA Cert for peer verification. |Yes

{% capture getEligibilityRequest %}
```shell
$ curl \
  --request GET \
  --url '{{site.webapiB2B}}/vehicle/{vin}' \
  --data-urlencode 'client_id=<client_id>' \
  --data-urlencode 'extension=onboardCapabilities' \
  --header 'x-introspect-realm: <realm>' \
  --key 'path/to/key.pem' \
  --cert 'path/to/client_cert.pem[:<cert_password>]' \
  --cacert 'path/to/ca_cert.pem' \
```
{% endcapture %}

{% capture getEligibilityResponse %}
```http
HTTP/1.1 200 OK
Date: Day, XX Mon YYYY HH:MM:SS GMT
Content-Type: application/json

{
  "createdAt": "2023-09-12T09:41:10.911Z",
  "vin": "VR1AB12C3D45678909",
  "motorization": "Thermic",
  "attributes": {...},
  "_embedded": {
    "extension": {
      "onboardCapabilities": {
        "data": [
          "data:telemetry", "telemetry:environment", "data:telemetry:privacy",
          "data:telemetry:vehicle", "data:telemetry:vehicle:ignition", "data:telemetry:vehicle:preconditioning",
          "data:telemetry:vehicle:energies", "data:telemetry:vehicle:engines", "data:telemetry:vehicle:doorsState",
          "data:telemetry:vehicle:powertrain", "data:telemetry:vehicle:battery", "data:telemetry:vehicle:safety",
          "data:telemetry:vehicle:odometer", "data:telemetry:vehicle:kinetic", "data:telemetry:vehicle:transmission",
          "data:telemetry:vehicle:adas", "data:telemetry:vehicle:lightingSystem", "data:telemetry:vehicle:maintenance",
          "data:telemetry:vehicle:drivingBehavior", "data:telemetry:vehicle:wipingBlades", "data:telemetry:vehicle:alarm",
          "data:position", "data:trip", "data:alert", "data:collision"
        ],
        "remote": {
          "operation": [
            "remote:door:write", "remote:preconditioning:write",
            "remote:horn:write", "remote:charging:write",
            "remote:lights:write", "remote:weakup:write", "remote:navigation:write"
          ],
          "details": {
            "charging": {
              "supported": true, 
              "immediate": { "start": true, "stop": true },
              "schedule": { "programs": {"supported": true,"size": 0}, "nextDelayedTime": true },
              "preferences": { "level": true, "type": true }
            },
            "preconditioning": { "supported": true, "programs": { "size": 0 } },
            "door": { "supported": true },
            "horn": { "supported": true },
            "lights": { "supported": true },
            "wakeup": { "supported": true },
            "navigation": { "supported": true }
          }
        }
      }
    }
  }
}
```
{% endcapture %}

{% assign page.subsection = "b2b" %}

{% include webapi-curl.md 
  apiEndpointB2C='/vehicle/{vin}'
  apiBaseUrl=site.webapiB2B
  httpVerb='GET'
  requestQuery=getEligibilityRequest
  200=getEligibilityResponse
%}

{% assign page.subsection = "b2c" %}

## 2️⃣ Request Consent

<img src="{{site.baseurl}}/assets/images/b2c-enroll-users.svg" alt="b2c-enroll-users" style="width:400px">

## 2️⃣•1️⃣ Authorization Request

In order to enroll an End User, you have to trigger the following HTTP authorization request from your App in the End User device. This process is called **OAuth2 Authorization Code Flow** and is widely defined in [RFC 6749](https://datatracker.ietf.org/doc/html/rfc6749).

This will prompt an external user agent (web view) where the End User will be able to log in and authorize access to its Stellantis account. The following parameters are required:

> **Scopes**: scopes are sets of data that a Ressource Owner can share to a third-party application through this authorization process. List of scope is [available here]({{site.baseurl}}/webapi/b2c/quickstart/data-availability-scopes/#list-of-scopes). You can also browse individual data and their scopes in the [Data Catalog]({{site.baseurl}}/connected-vehicles/data-catalog/#article).

Type|Name|Value|Description|Required
-|-|-|-|-
Path Parameter|`{brand.tld}`|`{brand.tld}`| Vehicle manufacturer [brand.tld](#manufacturers-brands--realms). | Yes
Query Parameter|`client_id`|`<client_id>`| [Client id]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#article) of your application.|Yes
Query Parameter|`response_type`|`code`| Use OAuth2 Authorization code flow. | Yes
Query Parameter|`scope`|`<scope1>,<scope2>,etc.`| The list of scopes requested to your customer (Ressource Owner). | Yes
Query Parameter|`redirect_uri`|`<redirect_uri>`| The URI address to redirect the user at the end of the authorization process. | Yes
Query Parameter|`state`|`<state>`| A state value generated on your side, it will be returned along with the [Authorization Code](#2️⃣•4️⃣-authorization-code) in the redirection. Recommended, checking this parameter when it's returned allows tracking and identifying the request, and prevent attacks. | No

{% capture getAuthRequest %}
```shell
$ curl \
  --request GET \
  --url '{{site.cvsOAuth2}}/authorize' \
  --data-urlencode 'client_id=<client_id>' \
  --data-urlencode 'response_type=code' \
  --data-urlencode 'scope=vehicle_read,remote_write' \
  --data-urlencode 'redirect_uri=<app_callback_uri>' \
  --data-urlencode 'state=<state>' \
```
{% endcapture %}


{% include webapi-curl.md 
  apiEndpointB2C='/authorize'
  apiBaseUrl=site.cvsOAuth2
  httpVerb='GET'
  requestQuery=getAuthRequest
  no_link=true
%}


## 2️⃣•2️⃣ Prompt Triggering

Before the Authorization prompt is triggered, Stellantis performs the following checks on the request:
- The **App should be registered** and the <client_id> should exists.
- The requested **scopes should match** the one declared in the [App Registration]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#article). 

In cases these checks are performed successfully, the redirection prompt is sent to the End User device triggering an **external user agent** (web view) on the End User device.

## 2️⃣•3️⃣ User Inputs

When the external user agent authorization prompt is triggered by the End User device, you have no more access to the authorization process. 

In the prompted external user agent (web view), the user will follow the **consent process** to give access to its Stellantis Account.

**User Journey**: if not previously performed for another Stellantis Connected service, the following steps are required on the End User side before being able to authorize access to its Stellantis Account:
- **Create an account** in Stellantis system.
- **Link a vehicle** to the account.
- **Enable connectivity** for partners (accessing party).

> **Closing the App:** If the user needs to leave your application during the previous process, the authorization request might fail. In this case, the user needs to **restart** the authorization process from the start.


## 2️⃣•4️⃣ Authorization Code

In the **OAuth2 Authorization Code Flow**, after the Resource Owner consent to share access to its data, an automatic redirection is triggered to the Accessing Party URI. This restriction includes an **Authorization Code** as an URI parameter. This Authorization Code can be exchanged against an **Access Token**.

If the account is **set up properly** (check out [step 2.3](#2️⃣3️⃣-user-inputs)), and the End-User **authorizes your App** to access the requested scopes, the redirection URI contains the **Authorization Code**.

The redirect URI will contain the following data as parameters:
- The list of **scopes authorized** by the user
- **State** value generated on your side from the initial request, for your own use.
- The **authorization code**, this code should be exchanged for an access token.

>**State:** This state is for your own use. In order to mitigate **CSRF** attacks, you should make sure that the state value received in 2.3 is a state value that you have generated in step 2.1.
<br>This state being generated on your side, you can also use it to **embed data** in step 2.1 for future use in step 2.3. Doing so, it's possible to **track the request** and to notify your user if a consent request has been initialized but not completed.

{% capture getAuthResponse %}
```http
HTTP/1.1 302 Found
Date: Day, XX Mon YYYY HH:MM:SS GMT
Location: http://<app_callback_uri>
          ?scope=ConnectivityForPartners,Remote
          &state=<state>
          &code=<code>
```
{% endcapture %}


{% include webapi-curl.md
  apiEndpointB2C='/authorize'
  apiBaseUrl=site.cvsOAuth2
  httpVerb='GET'
  hideRequest=true
  200=getAuthResponse
  no_link=true
%}

## 3️⃣ Access Token

<img src="{{site.baseurl}}/assets/images/b2c-request-access-token.svg" alt="b2c-request-access-token" style="width:525px">

The **Authentication Code** obtained in [step 2](#2️⃣-request-consent) should be exchanged against an **Access Token**.

> **Client Secret:** Exchanging the authentication code for an Access Token must be done from **your own server** and not from the Ressource Owner device. Furthermore the API used for this purpose requires your Application Client Secret and this data must remain confidential.

This API allows requesting access token, it requires the following parameters:

Type|Name|Value|Description|Required
-|-|-|-|-
Path Parameter|`{brand.tld}`|`{brand.tld}`| Vehicle manufacturer [brand.tld](#manufacturers-brands--realms). | Yes
Header|`authorization: Basic`|`<base64(<client_id>:<client_secret>)>`| Your App [Client Id and Secret]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#article) encoded in base64. |Yes
Header|`content-type`|`<application/x-www-form-urlencoded>`| Content type. |Yes
Data |`grant_type`|`authorization_code`| Use OAuth2 Authorization code flow. | Yes
Data |`redirect_uri`|`<redirect_uri>`| The URI redirection address. It must be the same as in step 2.1.| Yes
Data |`code`|`<code>`| The authorization code received in step 2.4. | Yes

{% capture postAccessTokenRequest %}
```shell
$ curl \
  --request POST \
  --url '{{site.cvsOAuth2}}/access_token' \
  --header 'authorization: Basic <base64(<client_id>:<client_secret>)>'\
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'grant_type=authorization_code' \
  --data 'redirect_uri=<app_callback_uri>' \
  --data 'code=<code>' \
```
{% endcapture %}

{% capture postAccessTokenResponse %}
```http
HTTP/1.1 200 OK
Date: Day, XX Mon YYYY HH:MM:SS GMT

{
  "scope": "openid profile",
  "expires_in": <time>,
  "token_type": "Bearer",
  "refresh_token": <refresh_token>,
  "id_token": <id_token>,
  "access_token": <access_token>
}
```
{% endcapture %}

{% include webapi-curl.md 
  apiEndpointB2C='/access_token'
  httpVerb='POST'
  apiBaseUrl=site.cvsOAuth2
  requestQuery=postAccessTokenRequest
  200=postAccessTokenResponse
  no_link=true
%}


## Renew Token

In case the access token expires, you should renew the set of tokens using the following API call from your server. 

This request requires that you still have access to the refresh token of [step 3](#3️⃣-access-token).

Type|Name|Value|Description|Required
-|-|-|-|-
Path Parameter|`{brand.tld}`|`{brand.tld}`| Vehicle manufacturer [brand.tld](#manufacturers-brands--realms). | Yes
Query Parameter | `realm`| `<realm>` | Vehicle manufacturer [realm](#manufacturers-brands--realms). | Yes 
Header|`authorization: Basic`|`<base64(<client_id>:<client_secret>)>`| Your App [Client id and secret]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#article) encoded in base64. |Yes
Header|`content-type`|`<application/x-www-form-urlencoded>`| Content type. |Yes
Data |`grant_type`|`refresh_token`| Token renewal. | Yes
Data |`refresh_token`|`<refresh_token>`| Refresh token received in step 3. | Yes

{% capture postRenewAccessTokenRequest %}
```shell
$ curl \
  --request POST \
  --url '{{site.cvsOAuth2}}/access_token' \
  --data-urlencode 'realm=<realm>'
  --header 'authorization: Basic <base64(<client_id>:<client_secret>)>'\
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'grant_type=refresh_token' \
  --data 'refresh_token=<refresh_token>' \
```
{% endcapture %}

{% capture postRenewAccessTokenResponse %}
```http
HTTP/1.1 200 OK
Date: Day, XX Mon YYYY HH:MM:SS GMT

{
    "scope": "openid profile",
    "expires_in": <time>,
    "token_type": "Bearer",
    "refresh_token": <refresh_token>,
    "id_token": <id_token>,
    "access_token": <access_token>
}
```
{% endcapture %}

{% include webapi-curl.md 
  apiEndpointB2C='/access_token'
  apiBaseUrl=site.cvsOAuth2
  httpVerb='POST'
  requestQuery=postRenewAccessTokenRequest
  200=postRenewAccessTokenResponse
  no_link=true
%}

## Revoke Token (Logout User)

Type|Name|Value|Description|Required
-|-|-|-|-
Path Parameter|`{brand.tld}`|`{brand.tld}`| Vehicle manufacturer [brand.tld](#manufacturers-brands--realms). | Yes
Query Parameter | `realm`| `<realm>` | Vehicle manufacturer [realm](#manufacturers-brands--realms). | Yes 
Header|`authorization: Basic`|`<base64(<client_id>:<client_secret>)>`| Your App [Client id and secret]({{site.baseurl}}/webapi/b2c/quickstart/app-registration/#article) encoded in base64. |Yes
Header|`content-type`|`<application/x-www-form-urlencoded>`| Content type. |Yes
Data |`token`|`<refresh_toke OR access_token>`| Access or refresh token. | Yes

You should implement a feature in your application if the End User wants to log out.

This action should lead to the revocation of the token:


{% capture postRevokeTokenRequest %}
```shell
$ curl \
  --request POST \
  --url '{{site.cvsOAuth2}}/token/revoke' \
  --data-urlencode 'realm=<realm>' \
  --header 'authorization: Basic <base64(<client_id>:<client_secret>)>'\
  --header 'content-type: application/x-www-form-urlencoded' \
  --data 'refresh_token=<refresh_toke OR access_token>' \
```
{% endcapture %}

{% capture postRevokeTokenResponse %}
```http
HTTP/1.1 200 OK
Date: Day, XX Mon YYYY HH:MM:SS GMT
```
{% endcapture %}

{% include webapi-curl.md 
  apiEndpointB2C='/token/revoke'
  apiBaseUrl=site.cvsOAuth2
  httpVerb='POST'
  requestQuery=postRevokeTokenRequest
  200=postRevokeTokenResponse
  no_link=true
%}

> **Note:** depending on your needs, it's possible to revoke the **access token** or the **renew token**. Revoking a refresh token implies the revocation of all associated access token.

#### Access User Data

Once you have an Access Token, you can [build the HTTP request]({{site.baseurl}}/webapi/b2c/quickstart/access-user-data/#article) to access this user data.