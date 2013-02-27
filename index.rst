A (very) short intro. to `mod_wsgi`
===================================

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

*   `WSGIDaemonProcess`_, to run app. in its own processes:

    .. code-block:: apache

        WSGIDaemonProcess mydomain-tld-myapp processes=2 threads=5 (...)

    .. note:: recommended by G.D. instead of default `embedded` mode

    Some of other options:

    *   ``processes`` & ``threads``:

        .. code-block:: apache

            WSGIDaemonProcess (...) processes=2 threads=5 (...)


.. _WSGIDaemonProcess:
    http://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIDaemonProcess
