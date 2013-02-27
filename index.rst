A (very) short intro. to `mod_wsgi`
===================================

Preliminaries
-------------

**Outline**:
    1.  a bit of context on serving `Python` web applications
    2.  some `mod_wsgi` configuration options

**Disclaimer**:
    .. note::

        I only use `gunicorn` for development,
        and a monolithic `mod_wsgi` configuration for prod. that "works for me"!


Serve a `Python` web app.
-------------------------

There are several ways to serve a `Python` web application:

*   standard `Python` library `wsgiref.simple_server`_
*   a dedicated `WSGI` server (`gunicorn`_, `waitress`_, `meinheld`_, etc.)
*   a full web server (`Apache`, `nginx`, etc.)


.. _wsgiref.simple_server:
        http://docs.python.org/2/library/wsgiref.html#module-wsgiref.simple_server
.. _gunicorn: http://gunicorn.org/
.. _waitress: http://docs.pylonsproject.org/projects/waitress/
.. _meinheld: http://meinheld.org/


Serving a `WSGI` app.
---------------------

Given a `WSGI`_ application, it is possible to serve it in multiple ways:

*   *development* and tests:
    a standalone `WSGI` server should suffice (e.g. `gunicorn`_, `waitress`_)

    .. code-block:: sh

        $ cd /var/www/philologic/caesar
        $ gunicorn dispatcher:philo_dispatcher

*   *production*: choose your stack, e.g.

    *   nginx + (gunicorn | uwsgi)
    *   [nginx +] apache + mod_wsgi


.. _WSGI: http://www.wsgi.org/


`Apache httpd` and `Python`
---------------------------

There are several ways to run some `Python` web application
on a `Apache` web server:

*   `CGI`, already used by `PhiloLogic4`
*   `mod_python`_
*   `mod_wsgi`_
*   `uwsgi`_
*   others?


.. _mod_python: http://www.modpython.org/
.. _mod_wsgi: http://code.google.com/p/modwsgi/
.. _uwsgi: http://projects.unbit.it/uwsgi/


`mod_wsgi`
----------

*   `Apache httpd` module
*   created and maintained by `Graham Dumpleton`_
*   feature full:

    *   mature + decent performance
    *   refresh app. without restart `Apache`, by touching `WSGI` file
    *   daemon mode, which could work with `virtualenv`_
    *   numerous configuration options

⇒ http://code.google.com/p/modwsgi/


.. _Graham Dumpleton: http://blog.dscpl.com.au/
.. _virtualenv: http://www.virtualenv.org/


Configuration (1)
-----------------

*   `WSGIScriptAlias`_, to link an URL to a `WSGI` app.

    .. code-block:: apache

        WSGIScriptAlias /myapp /path/to/my/app/application.wsgi

    should offers access to app. from ``http://my.domain.tld/myapp``

*   (probably needs) access to `WSGI` file:

    .. code-block:: apache

        <Directory /path/to/my/app>
            Order deny,allow
            Allow from all
        </Directory>


.. _WSGIScriptAlias:
        http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIScriptAlias


Configuration (2)
-----------------

*   `WSGIDaemonProcess`_, to run app. in its own process:

    .. code-block:: apache

        WSGIDaemonProcess mydomain-tld-myapp (...)

    .. note:: recommended by G.D. instead of default `embedded` mode

    Some of other options:

    *   ``processes`` & ``threads``:

        .. code-block:: apache

            WSGIDaemonProcess (...) processes=2 threads=5 (...)


.. _WSGIDaemonProcess:
    http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIDaemonProcess


Configuration (3)
-----------------

*   (`WSGIDaemonProcess` continued)

    *   ``user`` & ``group``:

        .. code-block:: apache

            WSGIDaemonProcess (...) user=work group=www-data (...)

    *   ``maximum-requests``:

        .. code-block:: apache

            WSGIDaemonProcess (...) maximum-requests=1000 (...)


Configuration (4)
-----------------

*   (`WSGIDaemonProcess` continued)

    *   ``python-path``:

        .. code-block:: apache

            WSGIDaemonProcess (...) \
                python-path=/usr/lib/python2.7,/usr/local/lib/python2.7/dist-packages,(...)

        which allows using a `virtualenv`!


Configuration (5)
-----------------

*   (`WSGIDaemonProcess` continued)

    *   link process group to  `WSGI` parent dir. by its *name*:

        .. code-block:: apache

            WSGIDaemonProcess mydomain-tld-myapp (...)

            <Directory /path/to/my/app>
                WSGIProcessGroup mydomain-tld-myapp
                WSGIApplicationGroup %{GLOBAL}
                (...)
            </Directory>


Complete example
----------------

.. code-block:: apache

    WSGIDaemonProcess mydomain-tld-myapp \
        processes=2 threads=5 \
        python-path=/usr/lib/python2.7,(...) \
        user=work group=www-data \
        maximum-requests=1000 \
        display-name=%{GROUP}

    <Directory /path/to/my/app>
        WSGIProcessGroup mydomain-tld-myapp
        WSGIApplicationGroup %{GLOBAL}
        Order deny,allow
        Allow from all
    </Directory>

    WSGIScriptAlias /myapp /path/to/my/app/application.wsgi


RTFantasticMaintainer's
-----------------------

*   `configuration guidelines
    <http://code.google.com/p/modwsgi/wiki/ConfigurationGuidelines>`_
*   `configuration directives
    <http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives>`_
*   `on virtualenv
    <http://code.google.com/p/modwsgi/wiki/VirtualEnvironments>`_
*   `FAQ
    <http://code.google.com/p/modwsgi/wiki/FrequentlyAskedQuestions>`_
*   `config issues
    <http://code.google.com/p/modwsgi/wiki/ConfigurationIssues>`_
    and `application issues
    <http://code.google.com/p/modwsgi/wiki/ApplicationIssues>`_
*   Graham Dumpleton:
    `its blog
    <http://blog.dscpl.com.au/search/label/mod_wsgi>`_,
    `some of its conferences
    <http://pyvideo.org/search?models=videos.video&q=graham+dumpleton>`_

