MusicBrainz Database Tools
==========================

SQLAlchemy Models
-----------------

If you are developing a Python application that needs access to the
`MusicBrainz <https://musicbrainz.org/>`__
`data <https://musicbrainz.org/doc/MusicBrainz_Database>`__, you can use
the ``mbdata.models`` module to get
`SQLAlchemy <http://www.sqlalchemy.org/>`__ models mapped to the
MusicBrainz database tables.

All tables from the MusicBrainz database are mapped, all foreign keys
have one-way relationships set up and some models, where it's essential
to access their related models, have two-way relationships (collections)
set up.

In order to work with the relationships efficiently, you should use the
appropriate kind of `eager
loading <http://docs.sqlalchemy.org/en/latest/orm/loading_relationships.html>`__.

Example usage of the models:

.. code:: python

    >>> from sqlalchemy import create_engine
    >>> from sqlalchemy.orm import sessionmaker
    >>> from mbdata.models import Artist
    >>> engine = create_engine('postgresql://musicbrainz:musicbrainz@127.0.0.1/musicbrainz', echo=True)
    >>> Session = sessionmaker(bind=engine)
    >>> session = Session()
    >>> artist = session.query(Artist).filter_by(gid='8970d868-0723-483b-a75b-51088913d3d4').first()
    >>> print artist.name

If you use the models in your own application and want to define foreign
keys from your own models to the MusicBrainz schema, you will need to
let ``mbdata`` know which metadata object to add the MusicBrainz tables
to:

.. code:: python

    from sqlalchemy.ext.declarative import declarative_base
    Base = declarative_base()

    # this should be the first place where you import anything from mbdata
    import mbdata.config
    mbdata.config.configure(base_class=Base)

    # now you can import and use the mbdata models
    import mbdata.models

You can also use ``mbdata.config`` to re-map the MusicBrainz schema
names, if your database doesn't follow the original structure:

.. code:: python

    import mbdata.config
    mbdata.config.configure(schema='my_own_mb_schema')

If you need sample MusicBrainz data for your tests, you can use
``mbdata.sample_data``:

.. code:: python

    from mbdata.sample_data import create_sample_data
    create_sample_data(session)

HTTP API
--------

**Note:** This is very much a work in progress. It is not ready to use
yet. Any help is welcome.

There is also a HTTP API, which you can use to access the MusicBrainz
data using JSON or XML formats over HTTP. This is useful if you want to
abstract away the MusicBrainz PostgreSQL database.

Installation:

.. code:: sh

    virtualenv --system-site-packages e
    . e/bin/activate
    pip install -r requirements.txt
    python setup.py develop

Configuration:

.. code:: sh

    cp settings.py.sample settings.py
    vim settings.py

Start the development server:

.. code:: sh

    MBDATA_API_SETTINGS=`pwd`/settings.py python -m mbdata.api.app

Query the API:

.. code:: sh

    curl 'http://127.0.0.1:5000/v1/artist/get?id=b10bbbfc-cf9e-42e0-be17-e2c3e1d2600d'

For production use, you should use server software like
`uWSGI <http://projects.unbit.it/uwsgi/>`__ and
`nginx <http://nginx.org/>`__ to run the service.

Solr Index
----------

Create a minimal Solr configuration:

.. code:: sh

    ./bin/create_solr_home.py -d /tmp/mbdata_solr

Start Solr:

.. code:: sh

    cd /path/to/solr-4.6.1/example
    java -Dsolr.solr.home=/tmp/mbdata_solr -jar start.jar

Development
-----------

Normally you should work against a regular PostgreSQL database with
MusicBrainz data, but for testing purposes, you can use a SQLite
database with small data sub-set used in unit tests. You can create the
database using:

.. code:: sh

    ./bin/create_sample_db.py sample.db

Then you can change your configuration:

.. code:: sh

    DATABASE_URI = 'sqlite:///sample.db'

Running tests:

.. code:: sh

    nosetests -v

If you want to see the SQL queries from a failed test, you can use the
following:

.. code:: sh

    MBDATA_DATABASE_ECHO=1 nosetests -v

Jenkins task that automatically runs the tests after each commit is
`here <http://build.oxygene.sk/job/mbdata/>`__.
