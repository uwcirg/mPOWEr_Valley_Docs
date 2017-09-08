# Setup of Valley Medical Center mPOWEr Dev Instance

## References
1. [cPRO Identity Provider (IdP) options for Authentication (OAuth, Shibboleth)](https://docs.google.com/document/d/1lsEueY4kwmDdr95N98KPhIwEOKOXWI6CqdZluPvhJ6c/edit?usp=sharing)


## Register App on App Orchard
1. Go to [App Orchard](apporchard.epic.com/Developer/Apps).
2. Create new app record.

## Create Domain Directory
1. ssh into `mpower-dev.cirg.washington.edu`.
2. create a new subdirectory in the following folder: `/srv/www/mpower-dev.cirg.washington.edu/htdocs`.

## Create MySQL User and Database
1. Log into phpMyAdmin at `https://mpower-dev.cirg.washington.edu/phpmyadmin`
2. Create `mpower_<instancename>` database.
3. Grant `mpower_dev` user access to `mpower_<instancename>` database.
4. Check out dhair2 trunk into Valley directory. (Use Git!)
5. Install Opauth plugin:
    ```
    cd <project_root>/app/Plugin
    git clone https://github.com/uwcirg/cakephp-opauth.git Opauth
    cd Opauth
    git submodule init
    git submodule update
    ```
5. Checkout SoF Strategy to `<project_root>/app/Plugin/Opauth/Strategy`:
    ```
    cd <project_root>/app/Plugin/Opauth/Strategy
    git clone https://github.com/uwcirg/opauth-sof.git Sof
    ```
6. Copy over the dhair_oauth config file: `cp <project_root>/app/Config/dhair/dhair_oauth{_ignore,}.php`
7. Update `dhair_oauth.php` with correct client id and client secret, which are available on the App Orchard App page.
8. Update `PatientsController.php` with special case for setting user. The default page in mPOWEr shows all patients. But if it is a provider, we want to take them directly to the patient's specific record. For future work, we need to clean this up, to allow for showing a list of all patients in some circumstances, and a specific patient in some :
  - Line ~1986 in the PatientsController
9. **Less File Problem** - code change in the last couple months looks for different file pattern name.
10. **Logging** - `app/temp/logs`


## TODO
1. Create staging using Git checkout. `assigned to pmanko`

## SoF OAuth Flow
![Overview](http://www.websequencediagrams.com/cgi-bin/cdraw?lz=RUhSIFNlc3Npb24gLT4-IEFwcDogUmVkaXJlY3QgdG8gaHR0cHM6Ly97YXBwIGxhdW5jaF91cml9P1xuAAgGPTEyMyZcbmlzcz0AIwlmaGlyIGJhc2UgdXJsfQpBcHAgLT4gRUhSIEZISVIgU2VydmVyOiBHRVQAVgoAJg4vbWV0YWRhdGEKACcPIC0AgR4HW0NvbmZvcm1hbmNlIHN0YXRlbWVudCBpbmNsdWRpbmcgT0F1dGggMi4wIGVuZHBvaW50IFVSTHNdAIEIBwCBCgZBdXRoegCBCAkAgWQVZWhyIGF1dGhvcml6AIFLBj9cbnNjb3BlPQCCCgYmXG4AewU9YWJjJgCCCA9hdWQ9AIIADyZcbi4uLgo&s=default)

### 1) Launch mPOWEr using Launch URL
Provide a `launch_url`. This url will be called by MyChart/Hyperspace when mPOWEr is launched. This behavior can be simulated in the App Orchard `Documentation > Launching your app from Epic > SMART on FHIR (OAuth2)` launch button.

![App Orchard Launch](http://www.clipular.com/c/5012200994635776.png?k=TAvIqkBuLGoUQp1Jp_X-NWWYtpI)

![App Orchard Launch](http://www.clipular.com/c/4601052902195200.png?k=xF7TXENbV1JjnIb_UfRITMaEMtc)

When launched, Epic redirects to `redirect_url` and provides the following two query string variables:

- `iss`: Identifies the EHR's FHIR endpoint, which the app can use to obtain additional details about the EHR, including its authorization URL.
- `launch`: Opaque identifier for this specific launch, and any EHR context associated with it. This parameter must be communicated back to the EHR at authorization time by passing along a launch=123 parameter.

#### HelloEpic Reference
Launch request is handled here: https://github.com/pmanko/HelloEpicRoR/blob/master/app/controllers/pages_controller.rb#L185

#### mPOWEr Reference
Launch request is handled here: https://github.com/uwcirg/opauth-sof/blob/master/SofStrategy.php#L79

### 2) Access `/metadata` URL and Parse Endpoint URLs

"The metadata response contains (among other details) the EHR’s capability statement identifying the OAuth authorize and token endpoint URLs for use in requesting authorization to access FHIR resources."

#### HelloEpic Reference
Metadata Accessed in Lines 191 - 208:
https://github.com/pmanko/HelloEpicRoR/blob/master/app/controllers/pages_controller.rb#L191

#### mPOWEr Reference
Metadata Access in `request` function, Lines 99-124:
https://github.com/uwcirg/opauth-sof/blob/master/SofStrategy.php#L99

### 3) Send Authorization Request
![Auth Flow](http://www.websequencediagrams.com/cgi-bin/cdraw?lz=bm90ZSBsZWZ0IG9mIEFwcDogUmVxdWVzdCBhdXRob3JpemF0aW9uCkFwcCAtPj4gRUhSIEF1dGh6IFNlcnZlcjogUmVkaXJlY3QgaHR0cHM6Ly97ZWhyADUJZV91cmx9Py4uLgoAZgVvdmVyADITQQAnCCBBcHBcbihtYXkgaW5jbHVkZSBlbmQtdXNlAE4GZW50aWMAgQ4FXG5hbmQADw4AgSYJKQpOb3RlIABWGE9uIGFwcHJvdmFsCgCBQRAgLT4-AIIBBwCBSBBhcHAgcgCBZwdfdXJpfT9jb2RlPTEyMyYAgVcJAII-DUV4Y2hhbmdlIGNvZGUgZm9yIGFjY2VzcyB0b2tlbjtcbmlmIGNvbmZpZGVudGlhbCBjbGllbnQsAIFyCXNlY3JldApBcHAtPgCCaBJQT1NUAIJsCgBPBSB1cmx9XG5ncmFudF90eXBlPQCDOg1fY29kZSYAgSQSAIJ7GwCCagdlIGEAgxQFAIEcFgCCaQcAg0YXSXNzdWUgbmV3AIFyBiB3aXRoIGNvbnRleHQ6XG4ge1xuIgCCEwZfAIIUBSI6IgCBcwYtAIIjBS14eXoiLFxuImV4cGlyZXMtaW4iOjM2MDAsXG4icGF0aWVudCI6IjQ1NiIsXG4uLi5cbn0Ag0MUAIVZBVsAgnYMIHJlc3BvbnNlXQ&s=default&h=NA3OIkJNCqFraI5a)

The request is sent to the `authorize_url` retrieved in the previous step.

The request needs to have the following params:
- `response_type`: This parameter must contain the value “code”.
- `client_id`: This parameter contains your web application’s client ID issued by Epic.
- `redirect_uri`: This parameter contains your application’s redirect URI. After the request completes on the Epic server, this URI will be called as a callback.
- `scope`: This parameter describes the information for which the web application is requesting access.

#### HelloEpic Reference
https://github.com/pmanko/HelloEpicRoR/blob/master/app/controllers/pages_controller.rb#L217

#### mPOWEr Reference
https://github.com/uwcirg/opauth-sof/blob/master/SofStrategy.php#L144

### 4) Auth Server Approval
If approved, the authorization server redirects to the redirect URL you supplied in the initial request with the following required query-string parameter:

`code`: This parameter contains the authorization code generated by Epic.

### 5) Use Code to Request Access Token/Secret  
After receiving the authorization code from the EHR’s authorization server, your application trades the code for an access token (JSON object) using HTTP POST to the authorization server. The following parameters are required in the POST:

- `grant_type`: In Epic's OAuth implementation, this parameter always contains the value authorization_code.
- `code`: This parameter contains the authorization code sent from Epic's authorization server to your application as a query parameter on the redirect URI as described above.
- `redirect_uri`: This parameter must contain the same redirect URI that you provided in the initial access request.
- `client_id`: This parameter must contain the application’s client ID issued by Epic that you provided in the initial request.

#### HelloEpic Reference
Lines 42 - 59: https://github.com/pmanko/HelloEpicRoR/blob/master/app/controllers/pages_controller.rb#L42

#### mPOWEr Reference
`oauth2callback` function, Lines 185 - 198:
https://github.com/uwcirg/opauth-sof/blob/master/SofStrategy.php#L185

### 6) Store Token/Secret
The authorization server responds to the HTTP POST request with a JSON object that includes an access token. The response contains the following parameters:

- `access_token`: This parameter contains the access token issued by Epic to your application and is used in future requests.
- `token_type`: In Epic's OAuth implementation, this parameter always includes the value bearer.
- `expires_in`: This parameter contains the number of seconds for which the access token is valid.
- `scope`: This parameter describes the access your application is authorized for.

#### HelloEpic Reference
https://github.com/pmanko/HelloEpicRoR/blob/master/app/controllers/pages_controller.rb#L62

#### mPOWEr Reference
https://github.com/uwcirg/opauth-sof/blob/master/SofStrategy.php#L206

## Opauth Strategy Overview
https://github.com/opauth/opauth/wiki/Strategy-contribution-guide

The only mandatory method for an Opauth strategy is `public function request()`. This is the function that handles the authentication with the authentication provider.

You are free to include other methods if the authentication process requires several round-trips. By declaring a method public, it will be reachable directly through HTTP via http://path_to_opauth/STRATEGY/METHOD_NAME.

## OAuth Scopes and User Identity
The implementation of SMART specifications in this area likely varies by EHR vendor. Determining the identity of the user logged in is important, since we need to differentiate between patients (MyChart Login) and clinicians (Hyperspace start).

From what was mentioned at FHIRWorks, it appears **Epic** does not really utilize scopes in any meaningful way. This is a good area to ask for clarification on, and figure out the current/future role of scopes. Here's a relevant 

### SMART Specs Overview
**The SMART specs give an overview of how to access user data:**

*Scopes for requesting identity data*

*Some apps need to authenticate the clinical end-user. This can be accomplished by requesting a pair of OpenID Connect scopes: openid and profile.*

*When these scopes are requested (and the request is granted), the app will receive an id_token that comes alongside the access token.*

*This token must be validated according to the OIDC specification. To learn more about the user, the app should treat the “profile” claim as the URL of a FHIR resource representing the current user. This will be a resource of type Patient, Practitioner, or RelatedPerson.*

**In addition, they provide information about the `User` scope:**

*User-level scopes allow access to specific data that a user can access. Note that this isn’t just data about the user; it’s data available to that user. User-level scopes take the form: user/:resourceType.(read|write|*).*

## mPOWEr Authentication Flow
