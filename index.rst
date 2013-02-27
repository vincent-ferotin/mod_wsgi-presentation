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

