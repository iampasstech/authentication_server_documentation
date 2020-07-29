.. _authentication_api:


.. toctree::
   :maxdepth: 2
   :caption: Authentication API

Authentication API
==================

The Activeconnect Authentication API is used to authenticate users.

**The client application is responsible for controlling access to protected resources**

For an overview of the authentication process see :ref:`getting_started`

Initiating the Authentication Process
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

   **URL**::
   https://activeapi.ninja/authentication/authenticate_user/<application_id>/<user_id>?methods=methods

   :param: application_id: The Application ID of the application.
   :type: application_id: string

   :param: user_id: The user to authenticate.
   :type: user_id: string

   :param: methods: (Optional) command separated list of authentication methods to use. If not present **(preferred)** Activeconnect will select appropriate methods.
   :type: application_id: string.

   :return: Authentication session data in json and http status code.

   **Example**::

        curl -X POST https://activeapi.ninja/authentication/authenticate_user/<application_id>/<user_id> -H 'cache-control: no-cache'


   **Expected Success Response**::

        HTTP Status Code 202 - Authentication Started

        HTTP Status Code 200 - Authentication did not start

        'authentication_status':
        {
            'authenticated': True/False,
            'session_status': (string) The status of the session
            'reason': (string) Failure reason
            'status_url': (string) The URL to call to get the session status
            'logout_url': (string) The URL to call to end the session
            'session_token': (string) A token that identifies the session
            'session_secret': (string) Secret used to authenticate calls to status_url and logout_url
        }



   **Session Status Values**

    Valid values for session status are defined in: :ref:`session_values`

   **Authentication Methods**

    Valid values for session status are defined in: :ref:`authentication_methods`

   **Expected Fail Response**::

        HTTP Status Code 404
        Client Application <ApplicationID> not found

   **Authentication**

        HMAC using ApplicationID and Application Secret


.. _session_values:

Session Status Values
^^^^^^^^^^^^^^^^^^^^^

    Values of session status are:
        * "pending" - the authentication is in progress
        * "timeout" - the authentication request has timed out (user did not respond)
        * "closed" - the session has been closed
        * "failed" - the authentication failed.
        * "walkaway" - the mobile device used for authentication is no longer nearby.
        * "active" - the user has been authenticated.
        * "identifying" - the request is being processed by a mobile device.
        * "cancelled" - the user cancelled the request.

    Any status other than:
        * "pending" - the authentication is in progress
        * "walkaway" - the mobile device used for authentication is no longer nearby.
        * "active" - the user has been authenticated.
        * "identifying" - the request is being processed by a mobile device.

    mean the user has not been authenticated or the session has ended.

.. _authentication_methods:

Authentication Methods
^^^^^^^^^^^^^^^^^^^^^^

    Current authentication methods are:
        * “acceptance” - user is prompted to confirm login attempt
        * “device “- user needs to unlock mobile device
        * “facial” - user needs perform facial recognition


Monitoring Session Status
^^^^^^^^^^^^^^^^^^^^^^^^^

   The response to calls to the authenticate_user endpoint contain a url to monitor the status of the session.

   **URL**::
   Contained in the status_url of the authenticate_user response BODY.


   :return: Authentication session status in json and http status code.

   **Authentication**
    Calls to this enpoint must use the *session_token* and *session_secret* to construct the authentication headers.

    See :ref:`hmac` for details.


   **Expected Success Response**::

        HTTP Status Code 200

        {
            'authenticated': True/False,
            'session_status': (string) The status of the session
        }



   **Session Status Values**

    Valid values for session status are defined in: :ref:`session_values`


   **Authentication**

        HMAC using *session_token* and *session_secret*.



Ending Sessions
^^^^^^^^^^^^^^^

   The response to calls to the authenticate_user endpoint contain a url to monitor the status of the session.
   **URL**::
   Contained in the logout of the authenticate_user response BODY.

   :return: Operation result in json and http status code.

   **Authentication**
    Calls to this enpoint must use the *session_token* and *session_secret* to construct the authentication headers.

    See :ref:`hmac` for details.


   **Expected Success Response**::

        HTTP Status Code 200

        {
            'status': True/False,
        }



   **Session Status Values**

    Valid values for session status are defined in: :ref:`session_values`


   **Authentication**:

        HMAC using *session_token* and *session_secret*.
