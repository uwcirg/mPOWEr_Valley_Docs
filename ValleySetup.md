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

### 1) Launch mPOWEr using Redirect URL
Provide a `redirect_url`, which is set in the App Orchard configuration. This url will be called by MyChart/Hyperspace when mPOWEr is launched. This behavior can be simulated in the App Orchard `Documentation > Launching your app from Epic > SMART on FHIR (OAuth2)` launch button.

![App Orchard Launch](http://www.clipular.com/c/5012200994635776.png?k=TAvIqkBuLGoUQp1Jp_X-NWWYtpI)

![App Orchard Launch](http://www.clipular.com/c/4601052902195200.png?k=xF7TXENbV1JjnIb_UfRITMaEMtc)

When launched, Epic redirects to `redirect_url` and provides the following two query string variables:

- `iss`: Identifies the EHR's FHIR endpoint, which the app can use to obtain additional details about the EHR, including its authorization URL.
- `launch`: Opaque identifier for this specific launch, and any EHR context associated with it. This parameter must be communicated back to the EHR at authorization time by passing along a launch=123 parameter.

#### HelloEpic Reference
Launch request is handled here: https://github.com/pmanko/HelloEpicRoR/blob/master/app/controllers/pages_controller.rb#L185

#### mPOWEr Reference
Launch request is handled here: https://github.com/uwcirg/opauth-sof/blob/master/SofStrategy.php#L79

###
