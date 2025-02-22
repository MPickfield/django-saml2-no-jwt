
# Django SAML2 Authentication


[![PyPI](https://img.shields.io/pypi/v/django-saml2-no-jwt?label=version&logo=pypi)](https://pypi.org/project/django-saml2-no-jwt/) [![GitHub Workflow Status](https://img.shields.io/github/actions/workflow/status/mpickfield/django-saml2-no-jwt/deploy.yml?branch=main&logo=github)](https://github.com/mpickfield/django-saml2-no-jwt/actions) [![Coverage Status](https://coveralls.io/repos/github/MPickfield/django-saml2-auth/badge.svg?branch=main)](https://coveralls.io/github/MPickfield/django-saml2-auth?branch=main)

This plugin provides a simple way to integrate SAML2 Authentication into your Django-powered app. SAML SSO is a standard, so practically any SAML2 based SSO identity provider is supported.

This plugin supports both identity provider and service provider-initiated SSO:

- For IdP-initiated SSO, the user should sign in to their identity provider platform, e.g., Okta, and click on the application that authorizes and redirects the user to the service provider, that is your platform.
- For SP-initiated SSO, the user should first exist on your platform, either by signing in via the first method (IdP-initiated SSO) or any other custom solution. It can be configured to be redirected to the correct application on the identity provider platform.

For IdP-initiated SSO, the user will be created if it doesn't exist. Still, for SP-initiated SSO, the user should exist in your platform for the code to detect and redirect them to the correct application on the identity provider platform.

## Project Information

This is a fork of https://github.com/grafana/django-saml2-auth @ v3.11 with pyjwt removed so that you can manage jwt yourself.

- Original Author: Fang Li ([@fangli](https://github.com/fangli))
- Maintainer: Mike Pickfield ([@mpickfield](https://github.com/mpickfield))
- Version support matrix:
  | **Python** | **Django** | **django-saml2-auth** | **End of Support<br/>(django-saml2-auth)** | **End of extended support<br/>(Django)** |
  | ---------------------------- | ---------- | --------------------- | ------------------------------------------ | ---------------------------------------- |
  | 3.7.x, 3.8.x, 3.9.x, 3.10.x | 2.2.x | >=3.4.0 | 3.10.0 | April 11, 2022 |
  | 3.7.x, 3.8.x, 3.9.x, 3.10.x | 3.2.x | >=3.4.0 | | April 2024 |
  | 3.8.x, 3.9.x, 3.10.x | 4.0.x | >=3.4.0 | 3.10.0 | April 1, 2023 |
  | 3.8.x, 3.9.x, 3.10.x | 4.1.x | >=3.4.0 | | December 2023 |
  | 3.8.x, 3.9.x, 3.10.x, 3.11.x | 4.2.x | >=3.4.0 | | April 2026 |

- Release logs are available [here](https://github.com/grafana/django-saml2-auth/releases).

- For contribution, read [contributing guide](CONTRIBUTING.md).

## CycloneDX SBOM

From [v3.6.1](https://github.com/grafana/django-saml2-auth/releases/tag/v3.6.1), CycloneDX SBOMs will be generated for [requirements.txt](./requirements.txt) and [requirements_test.txt](./requirements_test.txt) and it can be accessed from the latest build of GitHub Actions for a tagged release, for example, [this one](https://github.com/grafana/django-saml2-auth/actions/runs/2245422253). The artifacts are only kept for 90 days.

## Donate

Please give us a shiny ![star](https://img.shields.io/github/stars/grafana/django-saml2-auth.svg?style=social&label=Star&maxAge=86400) and help spread the word.

## Installation

You can install this plugin via `pip`. Make sure you update `pip` to be able to install from git:

```bash
pip install grafana-django-saml2-auth
```

or from source:

```bash
git clone https://github.com/grafana/django-saml2-auth
cd django-saml2-auth
python setup.py install
```

`xmlsec` is also required by `pysaml2`, so it must be installed:

```bash
// RPM-based distributions
# yum install xmlsec1
// DEB-based distributions
# apt-get install xmlsec1
// macOS
# brew install xmlsec1
```

[Windows binaries](https://www.zlatkovic.com/projects/libxml/index.html) are also available.

## How to use?

1. Once you have the library installed or in your `requirements.txt`, import the views module in your root `urls.py`:

   ```python
   import django_saml2_auth.views
   ```

2. Override the default login page in the root `urls.py` file, by adding these lines **BEFORE** any `urlpatterns`:

   ```python
   # These are the SAML2 related URLs. You can change "^saml2_auth/" regex to
   # any path you want, like "^sso/", "^sso_auth/", "^sso_login/", etc. (required)
   url(r'^sso/', include('django_saml2_auth.urls')),

   # The following line will replace the default user login with SAML2 (optional)
   # If you want to specific the after-login-redirect-URL, use parameter "?next=/the/path/you/want"
   # with this view.
   url(r'^accounts/login/$', django_saml2_auth.views.signin),

   # The following line will replace the admin login with SAML2 (optional)
   # If you want to specific the after-login-redirect-URL, use parameter "?next=/the/path/you/want"
   # with this view.
   url(r'^admin/login/$', django_saml2_auth.views.signin),
   ```

3. Add `'django_saml2_auth'` to `INSTALLED_APPS` in your django `settings.py`:

   ```python
   INSTALLED_APPS = [
       '...',
       'django_saml2_auth',
   ]
   ```

4. In `settings.py`, add the SAML2 related configuration:

   Please note, the only required setting is **METADATA_AUTO_CONF_URL** or the existence of a **GET_METADATA_AUTO_CONF_URLS** trigger function. The following block shows all required and optional configuration settings and their default values.

   ```python
   SAML2_AUTH = {
       # Metadata is required, choose either remote url or local file path
       'METADATA_AUTO_CONF_URL': '[The auto(dynamic) metadata configuration URL of SAML2]',
       'METADATA_LOCAL_FILE_PATH': '[The metadata configuration file path]',
       'KEY_FILE': '[The key file path]',
       'CERT_FILE': '[The certificate file path]',

       'DEBUG': False,  # Send debug information to a log file
       # Optional logging configuration.
       # By default, it won't log anything.
       # The following configuration is an example of how to configure the logger,
       # which can be used together with the DEBUG option above. Please note that
       # the logger configuration follows the Python's logging configuration schema:
       # https://docs.python.org/3/library/logging.config.html#logging-config-dictschema
       'LOGGING': {
           'version': 1,
           'formatters': {
               'simple': {
                   'format': '[%(asctime)s] [%(levelname)s] [%(name)s.%(funcName)s] %(message)s',
               },
           },
           'handlers': {
               'stdout': {
                   'class': 'logging.StreamHandler',
                   'stream': 'ext://sys.stdout',
                   'level': 'DEBUG',
                   'formatter': 'simple',
               },
           },
           'loggers': {
               'saml2': {
                   'level': 'DEBUG'
               },
           },
           'root': {
               'level': 'DEBUG',
               'handlers': [
                   'stdout',
               ],
           },
       },

       # Optional settings below
       'DEFAULT_NEXT_URL': '/admin',  # Custom target redirect URL after the user get logged in. Default to /admin if not set. This setting will be overwritten if you have parameter ?next= specificed in the login URL.
       'CREATE_USER': True,  # Create a new Django user when a new user logs in. Defaults to True.
       'NEW_USER_PROFILE': {
           'USER_GROUPS': [],  # The default group name when a new user logs in
           'ACTIVE_STATUS': True,  # The default active status for new users
           'STAFF_STATUS': False,  # The staff status for new users
           'SUPERUSER_STATUS': False,  # The superuser status for new users
       },
       'ATTRIBUTES_MAP': {  # Change Email/UserName/FirstName/LastName to corresponding SAML2 userprofile attributes.
           'email': 'user.email',
           'username': 'user.username',
           'first_name': 'user.first_name',
           'last_name': 'user.last_name',
           'groups': 'Groups',  # Optional
       },
       'GROUPS_MAP': {  # Optionally allow mapping SAML2 Groups to Django Groups
           'SAML Group Name': 'Django Group Name',
       },
       'TRIGGER': {
           # Optional: needs to return a User Model instance or None
           'GET_USER': 'path.to.your.get.user.hook.method',
           'CREATE_USER': 'path.to.your.new.user.hook.method',
           'BEFORE_LOGIN': 'path.to.your.login.hook.method',
           'AFTER_LOGIN': 'path.to.your.after.login.hook.method',
           # Optional. This is executed right before METADATA_AUTO_CONF_URL.
           # For systems with many metadata files registered allows to narrow the search scope.
           'GET_USER_ID_FROM_SAML_RESPONSE': 'path.to.your.get.user.from.saml.hook.method',
           # This can override the METADATA_AUTO_CONF_URL to enumerate all existing metadata autoconf URLs
           'GET_METADATA_AUTO_CONF_URLS': 'path.to.your.get.metadata.conf.hook.method',
           # jwt hooks
           'CUSTOM_CREATE_JWT': 'path.to.your.create.jwt.method',
           'CUSTOM_DECODE_JWT':'path.to.your.decode.jwt.method'
       },
       'ASSERTION_URL': 'https://mysite.com',  # Custom URL to validate incoming SAML requests against
       'ENTITY_ID': 'https://mysite.com/saml2_auth/acs/',  # Populates the Issuer element in authn request
       'NAME_ID_FORMAT': FormatString,  # Sets the Format property of authn NameIDPolicy element, e.g. 'user.email'
       'FRONTEND_URL': 'https://myfrontendclient.com',  # Redirect URL for the client if you are using JWT auth with DRF. See explanation below
       'LOGIN_CASE_SENSITIVE': True,  # whether of not to get the user in case_sentive mode
       'AUTHN_REQUESTS_SIGNED': True, # Require each authentication request to be signed
       'LOGOUT_REQUESTS_SIGNED': True,  # Require each logout request to be signed
       'WANT_ASSERTIONS_SIGNED': True,  # Require each assertion to be signed
       'WANT_RESPONSE_SIGNED': True,  # Require response to be signed
       'ACCEPTED_TIME_DIFF': None,  # Accepted time difference between your server and the Identity Provider
       'ALLOWED_REDIRECT_HOSTS': ["https://myfrontendclient.com"], # Allowed hosts to redirect to using the ?next parameter
       'TOKEN_REQUIRED': True,  # Whether or not to require the token parameter in the SAML assertion
   }
   ```

5. In your SAML2 SSO identity provider, set the Single-sign-on URL and Audience URI (SP Entity ID) to <http://your-domain/saml2_auth/acs/>

## How to debug?

To debug what's happening between the SAMLP Identity Provider and your Django application, you can use SAML-tracer for [Firefox](https://addons.mozilla.org/en-US/firefox/addon/saml-tracer/) or [Chrome](https://chrome.google.com/webstore/detail/saml-tracer/mpdajninpobndbfcldcmbpnnbhibjmch?hl=en). Using this tool, you can see the SAML requests and responses that are being sent back and forth.

Also, you can enable the debug mode in the `settings.py` file by setting the `DEBUG` flag to `True` and enabling the `LOGGING` configuration. See above for configuration examples.

_Note:_ Don't forget to disable the debug mode in production and also remove the logging configuration if you don't want to see internal logs of pysaml2 library.

## Module Settings

Some of the following settings are related to how this module operates. The rest are passed as options to the pysaml2 library. For more information on the pysaml2 library, see the [pysaml2 documentation](https://pysaml2.readthedocs.io/en/latest/howto/config.html), which contains examples of available settings. Also, note that all settings are not implemented in this module.

| **Field name**                          | **Description**                                                                                                                                                                                                                                                                                                                                                                                                                                             | **Data type(s)** | **Default value(s)**                                                                                                                     | **Example**                                              |
| --------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------- |
| **METADATA_AUTO_CONF_URL**              | Auto SAML2 metadata configuration URL                                                                                                                                                                                                                                                                                                                                                                                                                       | `str`            | `None`                                                                                                                                   | `https://ORG.okta.com/app/APP-ID/sso/saml/metadata`      |
| **METADATA_LOCAL_FILE_PATH**            | SAML2 metadata configuration file path                                                                                                                                                                                                                                                                                                                                                                                                                      | `str`            | `None`                                                                                                                                   | `/path/to/the/metadata.xml`                              |
| **KEY_FILE**                            | SAML2 private key file path                                                                                                                                                                                                                                                                                                                                                                                                                                 | `str`            | `None`                                                                                                                                   | `/path/to/the/key.pem`                                   |
| **CERT_FILE**                           | SAML2 public certificate file path                                                                                                                                                                                                                                                                                                                                                                                                                          | `str`            | `None`                                                                                                                                   | `/path/to/the/cert.pem`                                  |
| **DEBUG**                               | Send debug information to a log file                                                                                                                                                                                                                                                                                                                                                                                                                        | `bool`           | `False`                                                                                                                                  |                                                          |
| **LOGGING**                             | Logging configuration dictionary                                                                                                                                                                                                                                                                                                                                                                                                                            | `dict`           | Not set.                                                                                                                                 |                                                          |
| **DEFAULT_NEXT_URL**                    | Custom target redirect URL after the user get logged in. Default to /admin if not set. This setting will be overwritten if you have parameter `?next=` specificed in the login URL.                                                                                                                                                                                                                                                                         | `str`            | `admin:index`                                                                                                                            | `https://app.example.com/account/login`                  |
| **CREATE_USER**                         | Determines if a new Django user should be created for new users                                                                                                                                                                                                                                                                                                                                                                                             | `bool`           | `True`                                                                                                                                   |                                                          |
| **CREATE_GROUPS**                       | Determines if a new Django group should be created if the SAML2 Group does not exist                                                                                                                                                                                                                                                                                                                                                                        | `bool`           | `False`                                                                                                                                  |                                                          |
| **NEW_USER_PROFILE**                    | Default settings for newly created users                                                                                                                                                                                                                                                                                                                                                                                                                    | `dict`           | `{'USER_GROUPS': [], 'ACTIVE_STATUS': True, 'STAFF_STATUS': False, 'SUPERUSER_STATUS': False}`                                           |                                                          |
| **ATTRIBUTES_MAP**                      | Mapping of Django user attributes to SAML2 user attributes                                                                                                                                                                                                                                                                                                                                                                                                  | `dict`           | `{'email': 'user.email', 'username': 'user.username', 'first_name': 'user.first_name', 'last_name': 'user.last_name', 'token': 'token'}` | `{'your.field': 'SAML.field'}`                           |
| **TRIGGER**                             | Hooks to trigger additional actions during user login and creation flows. These `TRIGGER` hooks are strings containing a [dotted module name](https://docs.python.org/3/tutorial/modules.html#packages) which point to a method to be called. The referenced method should accept a single argument: a dictionary of attributes and values sent by the identity provider, representing the user's identity. Triggers will be executed only if they are set. | `dict`           | `{}`                                                                                                                                     |                                                          |
| **TRIGGER.GET_USER**                    | A method to be called upon getting an existing user. This method will be called before the new user is logged in and is used to customize the retrieval of an existing user record. This method should accept ONE parameter of user dict and return a User model instance or none.                                                                                                                                                                          | `str`            | `None`                                                                                                                                   | `my_app.models.users.get`                                |
| **TRIGGER.CREATE_USER**                 | A method to be called upon new user creation. This method will be called before the new user is logged in and after the user's record is created. This method should accept ONE parameter of user dict.                                                                                                                                                                                                                                                     | `str`            | `None`                                                                                                                                   | `my_app.models.users.create`                             |
| **TRIGGER.BEFORE_LOGIN**                | A method to be called when an existing user logs in. This method will be called before the user is logged in and after the SAML2 identity provider returns user attributes. This method should accept ONE parameter of user dict.                                                                                                                                                                                                                           | `str`            | `None`                                                                                                                                   | `my_app.models.users.before_login`                       |
| **TRIGGER.AFTER_LOGIN**                 | A method to be called when an existing user logs in. This method will be called after the user is logged in and after the SAML2 identity provider returns user attributes. This method should accept TWO parameters of session and user dict.                                                                                                                                                                                                               | `str`            | `None`                                                                                                                                   | `my_app.models.users.after_login`                        |
| **TRIGGER.GET_METADATA_AUTO_CONF_URLS** | A hook function that returns a list of metadata Autoconf URLs. This can override the `METADATA_AUTO_CONF_URL` to enumerate all existing metadata autoconf URLs.                                                                                                                                                                                                                                                                                             | `str`            | `None`                                                                                                                                   | `my_app.models.users.get_metadata_autoconf_urls`         |
| **TRIGGER.CUSTOM_DECODE_JWT**           | A hook function to decode the user JWT. This method will be called instead of the `decode_jwt_token` default function and should return the user_model.USERNAME_FIELD. This method accepts one parameter: `token`.                                                                                                                                                                                                                                          | `str`            | `None`                                                                                                                                   | `my_app.models.users.decode_custom_token`                |
| **TRIGGER.CUSTOM_CREATE_JWT**           | A hook function to create a custom JWT for the user. This method will be called instead of the `create_jwt_token` default function and should return the token. This method accepts one parameter: `user`.                                                                                                                                                                                                                                                  | `str`            | `None`                                                                                                                                   | `my_app.models.users.create_custom_token`                |
| **TRIGGER.CUSTOM_TOKEN_QUERY**          | A hook function to create a custom query params with the JWT for the user. This method will be called after `CUSTOM_CREATE_JWT` to populate a query and attach it to a URL; should return the query params containing the token (e.g., `?token=encoded.jwt.token`). This method accepts one parameter: `token`.                                                                                                                                             | `str`            | `None`                                                                                                                                   | `my_app.models.users.get_custom_token_query`             |
| **ASSERTION_URL**                       | A URL to validate incoming SAML responses against. By default, `django-saml2-auth` will validate the SAML response's Service Provider address against the actual HTTP request's host and scheme. If this value is set, it will validate against `ASSERTION_URL` instead - perfect for when Django is running behind a reverse proxy.                                                                                                                        | `str`            | `https://example.com`                                                                                                                    |                                                          |
| **ENTITY_ID**                           | The optional entity ID string to be passed in the 'Issuer' element of authentication request, if required by the IDP.                                                                                                                                                                                                                                                                                                                                       | `str`            | `None`                                                                                                                                   | `https://exmaple.com/sso/acs`                            |
| **NAME_ID_FORMAT**                      | Set to the string `'None'`, to exclude sending the `'Format'` property of the `'NameIDPolicy'` element in authentication requests.                                                                                                                                                                                                                                                                                                                          | `str`            | `<urn:oasis:names:tc:SAML:2.0:nameid-format:transient>`                                                                                  |                                                          |
| **USE_JWT**                             | Set this to the boolean `True` if you are using Django with JWT authentication                                                                                                                                                                                                                                                                                                                                                                              | `bool`           | `False`                                                                                                                                  |                                                          |
| **JWT_ALGORITHM**                       | JWT algorithm (str) to sign the message with: [supported algorithms](https://pyjwt.readthedocs.io/en/stable/algorithms.html).                                                                                                                                                                                                                                                                                                                               | `str`            | `HS512` or `RS512`                                                                                                                       |                                                          |
| **JWT_SECRET**                          | JWT secret to sign the message if an HMAC is used with the SHA hash algorithm (`HS*`).                                                                                                                                                                                                                                                                                                                                                                      | `str`            | `None`                                                                                                                                   |                                                          |
| **JWT_PRIVATE_KEY**                     | Private key (str) to sign the message with. The algorithm should be set to `RSA256` or a more secure alternative.                                                                                                                                                                                                                                                                                                                                           | `str` or `bytes` | `--- YOUR PRIVATE KEY ---`                                                                                                               |                                                          |
| **JWT_PRIVATE_KEY_PASSPHRASE**          | If your private key is encrypted, you must provide a passphrase for decryption.                                                                                                                                                                                                                                                                                                                                                                             | `str` or `bytes` | `None`                                                                                                                                   |                                                          |
| **JWT_PUBLIC_KEY**                      | Public key to decode the signed JWT token.                                                                                                                                                                                                                                                                                                                                                                                                                  | `str` or `bytes` | `'--- YOUR PUBLIC KEY ---'`                                                                                                              |                                                          |
| **JWT_EXP**                             | JWT expiry time in seconds                                                                                                                                                                                                                                                                                                                                                                                                                                  | `int`            | 60                                                                                                                                       |                                                          |
| **FRONTEND_URL**                        | If `USE_JWT` is `True`, you should set the URL to where your frontend is located (will default to `DEFAULT_NEXT_URL` if you fail to do so). Once the client is authenticated through the SAML SSO, your client is redirected to the `FRONTEND_URL` with the JWT token as `token` query parameter. Example: `https://app.example.com/?&token=<your.jwt.token`. With the token, your SPA can now authenticate with your API.                                  | `str`            | `admin:index`                                                                                                                            |                                                          |
| **AUTHN_REQUESTS_SIGNED**               | Set this to `False` if your provider doesn't sign each authorization request.                                                                                                                                                                                                                                                                                                                                                                               | `bool`           | `True`                                                                                                                                   |
| **LOGOUT_REQUESTS_SIGNED**              | Set this to `False` if your provider doesn't sign each logout request.                                                                                                                                                                                                                                                                                                                                                                                      | `bool`           | `True`                                                                                                                                   |                                                          |
| **WANT_ASSERTIONS_SIGNED**              | Set this to `False` if your provider doesn't sign each assertion.                                                                                                                                                                                                                                                                                                                                                                                           | `bool`           | `True`                                                                                                                                   |                                                          |
| **WANT_RESPONSE_SIGNED**                | Set this to `False` if you don't want your provider to sign the response.                                                                                                                                                                                                                                                                                                                                                                                   | `bool`           | `True`                                                                                                                                   |                                                          |
| **ACCEPTED_TIME_DIFF**                  | Sets the [accepted time diff](https://pysaml2.readthedocs.io/en/latest/howto/config.html#accepted-time-diff) in seconds                                                                                                                                                                                                                                                                                                                                     | `int` or `None`  | `None`                                                                                                                                   |                                                          |
| **ALLOWED_REDIRECT_HOSTS**              | Allowed hosts to redirect to using the `?next=` parameter                                                                                                                                                                                                                                                                                                                                                                                                   | `list`           | `[]`                                                                                                                                     | `['https://app.example.com', 'https://api.exmaple.com']` |

### Triggers

| **Setting name**                   | **Description**                                                                                                                             | **Interface**                                                                                |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------- |
| **GET_METADATA_AUTO_CONF_URLS**    | Auto SAML2 metadata configuration URL                                                                                                       | get_metadata_auto_conf_urls(user_id: Optional[str] = None) -> Optional[List[Dict[str, str]]] |
| **GET_USER_ID_FROM_SAML_RESPONSE** | Allows retrieving a user ID before GET_METADATA_AUTO_CONF_URLS gets triggered. Warning: SAML response still not verified. Use with caution! | get_user_id_from_saml_response(saml_response: str, user_id: Optional[str]) -> Optional[str]  |

### jwt token triggers

This is an example of the functions that could be passed to the `TRIGGER.CUSTOM_CREATE_JWT` (it uses the [DRF Simple JWT library](https://github.com/jazzband/djangorestframework-simplejwt/blob/master/docs/index.rst)) and to `TRIGGER.CUSTOM_TOKEN_QUERY`:

```python
from rest_framework_simplejwt.tokens import RefreshToken


def get_custom_jwt(user):
    """Create token for user and return it"""
    return RefreshToken.for_user(user)


def get_custom_token_query(refresh):
    """Create url query with refresh and access token"""
    return "?%s%s%s%s%s" % ("refresh=", str(refresh), "&", "access=", str(refresh.access_token))

```

## Customize Error Messages

The default permission `denied`, `error` and user `welcome` page can be overridden.

To override these pages put a template named 'django_saml2_auth/error.html', 'django_saml2_auth/welcome.html' or 'django_saml2_auth/denied.html' in your project's template folder.

If a 'django_saml2_auth/welcome.html' template exists, that page will be shown to the user upon login instead of the user being redirected to the previous visited page. This welcome page can contain some first-visit notes and welcome words. The [Django user object](https://docs.djangoproject.com/en/1.9/ref/contrib/auth/#django.contrib.auth.models.User) is available within the template as the `user` template variable.

To enable a logout page, add the following lines to `urls.py`, before any `urlpatterns`:

```python
# The following line will replace the default user logout with the signout page (optional)
url(r'^accounts/logout/$', django_saml2_auth.views.signout),

# The following line will replace the default admin user logout with the signout page (optional)
url(r'^admin/logout/$', django_saml2_auth.views.signout),
```

To override the built in signout page put a template named 'django_saml2_auth/signout.html' in your project's template folder.

If your SAML2 identity provider uses user attribute names other than the defaults listed in the `settings.py` `ATTRIBUTES_MAP`, update them in `settings.py`.

## For Okta Users

I created this plugin originally for Okta. The `METADATA_AUTO_CONF_URL` needed in `settings.py` can be found in the Okta Web UI by navigating to the SAML2 app's `Sign On` tab. In the `Settings` box, you should see:

    Identity Provider metadata is available if this application supports dynamic configuration.

The `Identity Provider metadata` link is the `METADATA_AUTO_CONF_URL`.

More information can be found in the [Okta Developer Documentation](https://developer.okta.com/docs/guides/saml-application-setup/overview/).

## Release Process

I adopted a reasonably simple release process, which is almost automated, except for two actions that needed to be taken to start a release:

1. Update [setup.py](setup.py) and increase the version number in the `setup` function. Unless something backward-incompatible is introduced, only the minor version is upgraded: 3.8.0 becomes 3.9.0.
2. Tag the `main` branch with the the `vSEMVER`, e.g. `v3.9.0`, and git-push the tag.
3. The release and publish to PyPI is handled in the CI/CD using GitHub Actions.
4. Create a new release with auto-generated (and polished) release notes on the tag.
5. Download [SBOM artifacts](https://github.com/grafana/django-saml2-auth/actions/runs/3227336576) generated by GitHub Actions for the corresponding run, and add them to the [release files](https://github.com/grafana/django-saml2-auth/releases/tag/v3.9.0).
