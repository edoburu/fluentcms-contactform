fluentcms-contactform
===================

A plugin for django-fluent-contents_ to show a simple contact form

Installation
============

First install the module, preferably in a virtual environment. It can be installed from PyPI::

    pip install fluentcms-contactform


Backend Configuration
---------------------

First make sure the project is configured for django-fluent-contents_.

Then add the following settings:

.. code-block:: python

    INSTALLED_APPS += (
        'fluentcms_contactform',
        'crispy_forms',    # for default template
    )

The database tables can be created afterwards::

    ./manage.py migrate

Now, the ``ContactFormPlugin`` can be added to your ``PlaceholderField``
and ``PlaceholderEditorAdmin`` admin screens.

Make sure the following settings are configured:

.. code-block:: python

    DEFAULT_FROM_EMAIL = '"Your Name" <you@example.org>'

    FLUENTCMS_CONTACTFORM_VIA = "Sitename"    # Will send a From: "Username via Sitename" email.

    IPWARE_META_PRECEDENCE_ORDER = (
        'REMOTE_ADDR',   # The HTTP header for IP address detection
    )


IP address detection
~~~~~~~~~~~~~~~~~~~~

The visitor IP-address is stored with the submited data.
The detection happens using django-ipware_ which uses a "it just works" approach.
For security, make sure the correct HTTP header is checked for IP addresses.

For default sites (Apache + mod_wsgi / Nginx + uWSGI) add the following to your settings:

.. code-block:: python

    IPWARE_META_PRECEDENCE_ORDER = (
        'REMOTE_ADDR',
    )

When the WSGI proces runs as a separate HTTP server (for Gunicorn),
or runs behind a load balancer (HAProxy or Nginx reverse proxy), configure the following:

.. code-block:: python

    IPWARE_META_PRECEDENCE_ORDER = (
        'HTTP_X_FORWARDED_FOR',
    )

If your site has it's own IP resolver function, you can also configure it. The default is:

.. code-block:: python

    FLUENTCMS_CONTACTFORM_IP_RESOLVER = 'ipware.ip.get_real_ip'


Updating the form layout
~~~~~~~~~~~~~~~~~~~~~~~~

The default form fields can be changed using:

.. code-block:: python

    FLUENTCMS_CONTACTFORM_DEFAULT_FIELDS = ('name', 'email', 'phone_number', 'subject', 'message')

The form styles can be defined using:

.. code-block:: python

    FLUENTCMS_CONTACTFORM_STYLES = (
        ('default', {
            'title': _("Default"),
            'form_class': 'fluentcms_contactform.forms.default.ContactForm',
        }),
        ('captcha', {
            'title': _("Default with captcha"),
            'form_class': 'fluentcms_contactform.forms.captcha.CaptchaContactForm',
        }),
    )

You can provide any form class, as long as it inherits from ``fluentcms_contactform.forms.AbstractContactForm``.
The current implementation expects the form to be a model form,
so any submitted data is safely stored in the database too.


Displaying phone numbers
~~~~~~~~~~~~~~~~~~~~~~~~

The phone number field uses django-phonenumber-field_ to validate the phone number.
By default, it requires an international notation starting with ``+``.
The ``PhoneNumberField`` can support national phone numbers too, 
which is useful when most visitors come from a single country.
Update the ``PHONENUMBER_DEFAULT_REGION`` setting to reflect this.

For example, to auto insert a ``+31`` prefix for Dutch phone numbers, use:

.. code-block:: python

    PHONENUMBER_DEFAULT_REGION = 'NL'   # Your country code, eg. .NL to 

The phone numbers can be displayed in various formats, the most human readable is:

.. code-block:: python

    PHONENUMBER_DEFAULT_FORMAT = 'NATIONAL'

The supported formats are:

* ``NATIONAL`` - nicely space separated, remove the country prefix.
* ``INTERNATIONAL`` - nicely space separated
* ``E164`` - all numbers, suitable for data transmission.
* ``RFC3966`` - the ``tel:`` URL, suitable for URL display.


Frontend Configuration
----------------------

If needed, the HTML code can be overwritten by redefining ``fluentcms_contactform/forms/*.html``.

The template filename corresponds with the form style defined in ``FLUENTCMS_CONTACTFORM_STYLES``.
When no custom template is defined, ``fluentcms_contactform/forms/default.html`` will be used.

The staff email message can be updated by redefining ``fluentcms_contactform/staff_email/*.txt``,
which works similar to the form templates.


Contributing
------------

If you like this module, forked it, or would like to improve it, please let us know!
Pull requests are welcome too. :-)

.. _django-fluent-contents: https://github.com/edoburu/django-fluent-contents
.. _django-phonenumber-field: https://github.com/stefanfoulis/django-phonenumber-field
.. _django-ipware: https://github.com/un33k/django-ipware