.. _management_api:

.. toctree::
   :maxdepth: 2
   :caption: Management API


Management API
==============

The Activeconnect Management API is used to manage your *Activeconnect Applications*
You use the Management API to:
   * Add users
   * Delete users
   * Registser mobile devices
   * Detecting if users have registered a mobile device
   * Handling lost devices

All calls to the Activeconnect API must include authentication data as described in :ref:`hmac`



.. _adding-users-label:
Adding Users
------------
   **URL**::
   https://activeapi.ninja/add_users/<application_id>

   :param: application_id: The Application ID of the application.
   :type: application_id: string

   :return: The added users in json and http status code

   **Example**::

        curl -X POST https://activeapi.ninja/management/add_users/<application_id> -H 'cache-control: no-cache' -H 'content-type: application/json' \
        -d '{
            'users': ['user1', 'user2']
        }'

   **Expected Success Response**::

        HTTP Status Code 201

        {
            'status': True,
            'users': {
                'created': ['user1', 'user2'],
                'existing': ['existing-user1']
            }
        }

        HTTP Status 200

        {
            'status': False,
            'reason': (string)
        }

   **Expected Fail Response**::

        HTTP Status Code 404

        Client Application <application_id> not found

   **Authentication**

        HMAC using Application ID and Application Secret


Deleting Users
--------------
   **URL**::
   https://activeapi.ninja/delete_users/<application_id>

   :param: application_id: The Application ID of the application.
   :type: application_id: string


   :return: Operation result as json and HTTP status code

   **Example**::

      curl -X POST https://activeapi.ninja/management/delete_users/<application_id> -H 'cache-control: no-cache' -H 'content-type: application/json' \
      -d '{
          'users': ['user1', 'user2']
      }'

   **Expected Success Response**::

      HTTP Status Code 200

      {
          'status': True,
      }

   **Expected Fail Response**:

   HTTP Status Code 404

   Client Application <application_id> not found

   **Authentication**

   HMAC using Application ID and Application Secret


Device Registration
-------------------
   This endpoint will obtain a device registration link that can be shared with users.

   See :ref:`getting_started` for information about registration links.

   **URL**::
   https://activeapi.ninja/device_registration_link/application_id/user_id?display_name=display_name

   :param: application_id: The Application ID of the application.
   :type: application_id: string

   :param: user_id: The ID of the user. This value must be URL encoded.
   :type: user_id: string

   :param: display_name: (Optional) string that will be used by the Activeconnect Mobile App to display the user information. You can use something like *'user1@my_application'*. This value must be URL encoded.


   :return: Operation result as json and HTTP status code

   **Example**::

      curl -X GET https://activeapi.ninja/management/device_registration_link/<application_id>/<userID>?display_name="user1" -H 'cache-control: no-cache' -H 'content-type: application/json'


   **Expected Success Response**::

      HTTP Status Code 200

      {
          'register_url': (string)
          'status': True,
      }

   **Expected Fail Response**:

   HTTP Status Code 404

   Client Application *application_id*  or User *user_id* not found.

   **Authentication**

   HMAC using Application ID and Application Secret


Checking for Registered Device
------------------------------

   **URL**::
   https://activeapi.ninja/has_registered_mobile_device/application_id/user_id

   :param: application_id: The Application ID of the application.
   :type: application_id: string

   :param: user_id: The ID of the user. This value must be URL encoded.
   :type: user_id: string

   :return: Operation result as json and HTTP status code

   **Example**::

      curl -X GET https://activeapi.ninja/management/has_registered_mobile_device/<application_id>/<userID> -H 'cache-control: no-cache' -H 'content-type: application/json'


   **Expected Success Response**::

      HTTP Status Code 200

      {
          'device_registered': True/False
          'status': True,
      }

   **Expected Fail Response**:

   HTTP Status Code 404

   Client Application *application_id*  or User *user_id* not found.

   **Authentication**

   HMAC using Application ID and Application Secret


Dealing with Lost Devices
-------------------------
   This endpoint will disable a user's mobile device and generate a new registration link.

   See :ref:`getting_started` for information about registration links.

   **URL**::
   https://activeapi.ninja/lost_user_mobile_device/application_id/user_id

   :param: application_id: The Application ID of the application.
   :type: application_id: string

   :param: user_id: The ID of the user. This value must be URL encoded.
   :type: user_id: string

   :return: Operation result as json and HTTP status code

   **Example**::

      curl -X GET https://activeapi.ninja/management/lost_user_mobile_device/<application_id>/<userID> -H 'cache-control: no-cache' -H 'content-type: application/json'


   **Expected Success Response**::

      HTTP Status Code 200

      {
          'register_url': (string)
          'status': True,
      }

   **Expected Fail Response**:

   HTTP Status Code 404

   Client Application *application_id*  or User *user_id* not found.

   **Authentication**

   HMAC using Application ID and Application Secret