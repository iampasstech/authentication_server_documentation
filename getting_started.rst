.. _getting_started:


.. toctree::
   :maxdepth: 2
   :caption: Getting Started

Getting Started
===============

Introduction
############
The first stage in integrating Activeconnect into your application is to create an account using the `Activeconnect Console. <https://activeconnect.activeapi.ninja>`__

Creating your First Application
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    * Once you have completed the registration process you are ready to create your first application.
    * **Activeconnect Applications** are how you connect your application to Activeconnect.
    * Click the **'ADD APPLICATION'** button on the Activeconnect Console and enter a name for your application.
    * Once the application is created you will see a confirmation dialog, that contains the credentials your application will use to authenticate with the Activeconnect API.
    * **This is the only time** you will be able to see the credentials.

Example Application
^^^^^^^^^^^^^^^^^^^
If you prefer to read code rather than documentation you can check out our `Example Application. <https://github.com/Activeconnect/microblog>`__
This is a Fork of `Miguel Grinberg's Flask Mega-Tutorial. <https://blog.miguelgrinberg.com/post/the-flask-mega-tutorial-part-i-hello-world>`__

Managing your Application
^^^^^^^^^^^^^^^^^^^^^^^^^
The Activeconnect: :ref:`management_api` is used to manage your application.

.. _adding-users-guide-label:
Adding Users
^^^^^^^^^^^^
Before you can authenticate a user, you have to register the user with Activeconnect.
To do this use the add_users route.

We recommend that you create a lookup table in your application that associates your Users, with a token used to authenticate with Activeconnect

In the example below, the application has a **User** table that contains a **name** column and an **Activeconnect Token** table that contains a **user_id** column.
The **user_id** column has a foreign key constraint to **User.id**

The **token** field should be used as the id passed to the add_user route.

.. table:: User Table

+-------------------+
| User Table        |
+-----+-------------+
| id  |     name    |
+=====+=============+
|  1  |   user1     |
+-----+-------------+
|  2  |   user2     |
+-----+-------------+

.. table:: Lookup Table

+-------------------------------------------------------+
| Activeconnect Token Table                             |
+-----+-------------------------+-----------------------+
| id  |   user_id (FK User id)  |   token               |
+=====+=========================+=======================+
|  1  |   1                     |  token1               |
+-----+-------------------------+-----------------------+
|  2  |   2                     |  token2               |
+-----+-------------------------+-----------------------+


Registering Mobile Devices
^^^^^^^^^^^^^^^^^^^^^^^^^^

Before a user can authenticate using Activeconnect, they have to register a mobile device.

To register a mobile device, the user must be provided with a registration link to open on their phone.
We leave the decision as to how to share the link with your users up to you as Activeconnect does not store **any contact information** for your users.

To obtain a registration link, use the Management API **device_registration_link** route.
Once you have the device registration link, you can:
    * Send it in an email
    * Send it in an SMS message (this will ensure the link is sent to a mobile device)
    * Render a QR code in your application. The Activeconnect mobile applications have the ability to scan QR codes and register devices.


Checking if a user has Registered a Mobile Device
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you need to know whether a user has registered a mobile device, you can use the Management API **has_registered_mobile_device** route.


Authenticating Users
^^^^^^^^^^^^^^^^^^^^

Authentication is handled by the Activeconnect Authentication API :ref:`authentication_api`

Activeconnect authentication is an asynchronous process
    * Call the **authenticate_user** route and store the returned data.
    * Call the **status_url** endpoint in the authentication data until authentication completes (or fails)

Monitoring Status
^^^^^^^^^^^^^^^^^

Activeconnect provides the ability to
    * Remotely log out users.
    * Log users out if they leave the area where they logged in.

If you want your application to respond to these events, you should periodically call the **status_url** of the authentication data.
You can then update your application state based on the response.


Logging Users Out
^^^^^^^^^^^^^^^^^

The **authenticate_user** route returns information that identifies the session. To end an Activeconnect session call the **logout_url** route in this data.
