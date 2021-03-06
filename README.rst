RESTinstance
============

`Robot Framework <http://robotframework.org>`__ test library for (RESTful) JSON APIs

.. image:: https://circleci.com/gh/asyrjasalo/RESTinstance.svg?style=svg
    :target: https://circleci.com/gh/asyrjasalo/RESTinstance



Advantages
----------

1. **RESTinstance relies on Robot Framework's language-agnostic,
   clean and minimal syntax, for API tests.** It is neither tied to any
   particular programming language nor development framework.
   Using RESTinstance requires little, if any, programming knowledge.
   It builts on long-term technologies with well established communities,
   such as HTTP, JSON (Schema), Swagger/OpenAPI and Robot Framework.

2. **It validates JSON using JSON Schema, guiding you to write API tests
   to base on properties** rather than on specific values (e.g. "email
   must be valid" vs "email is foo\@bar.com"). This approach reduces test
   maintenance when the values responded by the API are prone to change.
   Although values are not required, you can still test them whenever they
   make sense (e.g. GET response body from one endpoint, then POST some
   of its values to another endpoint and verify the results).

3. **It generates JSON Schema for requests and responses automatically,
   and the schema gets more accurate by your tests.**
   Output the schema to a file and reuse it as expectations to test the other
   methods, as most of them respond similarly with only minor differences.
   Or extend the schema further to a full Swagger spec (version 2.0,
   OpenAPI 3.0 also planned), which RESTinstance can test requests and
   responses against. All this leads to reusability, getting great test
   coverage with minimum number of keystrokes and very clean tests.



Installation
------------

Pick the one that suits your environment best.

As a Python package
~~~~~~~~~~~~~~~~~~~
On 3.6, 3.7 and 2.7, you can install and upgrade `from PyPi <https://pypi.org/project/RESTinstance>`__:

::

    pip install --upgrade RESTinstance

This also installs `Robot Framework <https://pypi.org/project/robotframework>`__ if you do not have it already.

As a Docker image
~~~~~~~~~~~~~~~~~

`RESTinstance Docker image <https://hub.docker.com/r/asyrjasalo/restinstance/tags>`__
contains Python 3.6 and `the latest Robot Framework <https://pypi.org/project/robotframework/3.1.1>`__:

::

    docker pull asyrjasalo/restinstance



Usage
-----

There is a `step-by-step tutorial <https://github.com/asyrjasalo/RESTinstance/blob/master/examples>`__
in the making, best accompanied with `keyword documentation <https://asyrjasalo.github.io/RESTinstance>`__.

Quick start
~~~~~~~~~~~

1. Create two new (empty) directories ``tests`` and ``results``.

2. Create a new file ``tests/YOURNAME.robot`` with content:

.. code:: robotframework

    *** Settings ***
    Library         REST    https://jsonplaceholder.typicode.com
    Documentation   Test data can be read from variables and files.
    ...             Both JSON and Python type systems are supported for inputs.
    ...             Every request creates a so-called instance. Can be `Output`.
    ...             Most keywords are effective only for the last instance.
    ...             Initial schemas are autogenerated for request and response.
    ...             You can make them more detailed by using assertion keywords.
    ...             The assertion keywords correspond to the JSON types.
    ...             They take in either path to the property or a JSONPath query.
    ...             Using (enum) values in tests optional. Only type is required.
    ...             All the JSON Schema validation keywords are also supported.
    ...             Thus, there is no need to write any own validation logic.
    ...             Not a long path from schemas to full Swagger/OpenAPI specs.
    ...             The persistence of the created instances is the test suite.
    ...             Use keyword `Rest instances` to output the created instances.


    *** Variables ***
    ${json}         { "id": 11, "name": "Gil Alexander" }
    &{dict}         name=Julie Langford


    *** Test Cases ***
    GET an existing user, notice how the schema gets more accurate
        GET         /users/1                  # this creates a new instance
        Output schema   response body
        Object      response body             # values are fully optional
        Integer     response body id          1
        String      response body name        Leanne Graham
        [Teardown]  Output schema             # note the updated response schema

    GET existing users, use JSONPath for very short but powerful queries
        GET         /users?_limit=5           # further assertions are to this
        Array       response body
        Integer     $[0].id                   1           # first id is 1
        String      $[0]..lat                 -37.3159    # any matching child
        Integer     $..id                     maximum=5   # multiple matches
        [Teardown]  Output  $[*].email        # outputs all emails as an array

    POST with valid params to create a new user, can be output to a file
        POST        /users                    ${json}
        Integer     response status           201
        [Teardown]  Output  response body     ${OUTPUTDIR}/new_user.demo.json

    PUT with valid params to update the existing user, values matter here
        PUT         /users/2                  { "isCoding": true }
        Boolean     response body isCoding    true
        PUT         /users/2                  { "sleep": null }
        Null        response body sleep
        PUT         /users/2                  { "pockets": "", "money": 0.02 }
        String      response body pockets     ${EMPTY}
        Number      response body money       0.02
        Missing     response body moving      # fails if property moving exists

    PATCH with valid params, reusing response properties as a new payload
        &{res}=     GET   /users/3
        String      $.name                    Clementine Bauch
        PATCH       /users/4                  { "name": "${res.body['name']}" }
        String      $.name                    Clementine Bauch
        PATCH       /users/5                  ${dict}
        String      $.name                    ${dict.name}

    DELETE the existing successfully, save the history of all requests
        DELETE      /users/6                  # status can be any of the below
        Integer     response status           200    202     204
        Rest instances  ${OUTPUTDIR}/all.demo.json  # all the instances so far


3. Chose Python installation? Let's go (not that language):

::

    robot --outputdir results tests/

If you chose the Docker method instead (recall the story about red and blue pill here, if you want), this is quaranteed to work in most environments:

::

    docker run --rm -ti --env HOST_UID=$(id -u) --env HOST_GID=$(id -g) \
      --env HTTP_PROXY --env HTTPS_PROXY --network host \
      --volume "$PWD/tests":/home/robot/tests \
      --volume "$PWD/results":/home/robot/results \
      asyrjasalo/restinstance tests/

Tip: If you prefer installing from source, ``pip install --editable .``
and verify the installation with ``robot README.rst``



Contributing
------------

Bug reports and feature requests are tracked in
`GitHub <https://github.com/asyrjasalo/RESTinstance/issues>`__.

We do respect pull request(er)s. Please mention if you do not want to be
listed below as contributors.

A `CircleCI <https://circleci.com/gh/asyrjasalo/RESTinstance>`__ job is
created automatically for your GitHub pull requests as well.


Local development
~~~~~~~~~~~~~~~~~
On Linux distros and on OS X, may ``make`` rules ease repetitive workflows:

::

    $ make help
    all                  Run test, build, install and atest (default)
    atest                Run acceptance tests
    atest_py2            Run acceptance tests on Python 2
    black                Reformat source code in-place
    build                Build source dist and wheel
    check-manifest       Run check-manifest for MANIFEST.in completeness
    clean                Remove .venvs, builds, dists, and caches
    dc                   Start docker-composed test API on background
    dc_rm                Stop and remove docker-composed test API
    flake8               Run flake8 for static code analysis
    install              Install package from source tree, as --editable
    install_pypi         Install the latest PyPI release
    install_test         Install the latest test.pypi.org release
    libdoc               Regenerate library keyword documentation
    mypy                 Run mypy for static type checking
    publish_pypi         Publish dists to PyPI
    publish_test         Publish dists to test.pypi.org
    pur                  Update requirements-dev for locked versions
    pyroma               Run pyroma for Python packaging best practices
    retest               Run failed tests only, if none, run all
    test                 Run tests, installs requirements(-dev) first
    uninstall            Uninstall the package, regardless of its origin

Running ``make`` runs rules ``test``, ``build``, ``install`` and ``atest``
at once, and uses separate virtualenvs ``.venvs/dev/`` and ``.venvs/release/``
to ensure that no (user or system level) dependencies interfere with the
process.

If ``make`` is not available, you can setup for development with:

::

    virtualenv --no-site-packages .venvs/dev
    source .venvs/dev/bin/activate
    pip install --editable .

To recreate the keyword documentation from source (equals to ``make libdoc``):

::

    python -m robot.libdoc REST docs/index.html


Acceptance tests
~~~~~~~~~~~~~~~~

The ``testapi/`` is built on `mountebank <https://www.mbtest.org>`__.
You can monitor requests and responses at
`localhost:2525 <http://localhost:2525/imposters>`__

To start it with ``docker-compose`` (daemonized) and run all acceptance tests:

::

    make atest

This uses ``rfdocker`` underneath to build a yet another container for tests.
Host directories ``tests/`` and ``results/`` are accessed inside the container
via the respective Docker volumes. Same arguments are accepted as for ``robot``.

To run only specific test suite(s):

::

    RUN_ARGS="--network=host --env HTTP_PROXY --env HTTPS_PROXY" ./rfdocker tests/output.robot

Host network is used to minimize divergence between different host OSes.
It may or may not be necessary to pass any of ``RUN_ARGS`` in your environment,
but there should be no downside either (on OS X ``--network=host`` is required).

If Docker (Compose) is not available, you can use npm's ``npx`` to install
`mountebank npm package <https://www.npmjs.com/package/mountebank>`__
and start the very same test API (keep ``--localOnly`` for security):

::

    npx mountebank --localOnly  --allowInjection --configfile testapi/apis.ejs

And run tests on Python:

::

    python -m robot --outputdir results tests/


Docker releases
~~~~~~~~~~~~~~~

`The Docker image <https://hub.docker.com/r/asyrjasalo/restinstance/tags>`__
is built with (included) `rfdocker <https://github.com/asyrjasalo/rfdocker>`__
(regarding the changed parts) each time ``make atest`` is run.

To push it to your Docker registry as "latest" (remember to ``docker login``):

::

    ./release_docker https://your.docker.registry.com/restinstance

For `Docker Hub <https://hub.docker.com>`__, just org/username will do:

::

    ./release_docker {{organization}}/restinstance



Credits
-------

RESTinstance is under `Apache License 2.0 <https://github.com/asyrjasalo/RESTinstance/blob/master/LICENSE>`__
and was originally written by `Anssi Syrjäsalo <https://github.com/asyrjasalo>`__.

It was first presented at the first `RoboCon <https://robocon.io>`__, 2018.


Contributors:

- `jjwong <https://github.com/jjwong>`__
  for helping with keyword documentation and examples (also check
  `RESTinstance_starter_project <https://github.com/jjwong/RESTinstance_starter_project>`__)

- `Przemysław "sqilz" Hendel <https://github.com/sqilz>`__
  for using and testing RESTinstance in early phase (also check
  `RESTinstance-wrapper <https://github.com/sqilz/RESTinstance-wrapper>`__)

- `Vinh "vinhntb" Nguyen <https://github.com/vinhntb>`__, `#52 <https://github.com/asyrjasalo/RESTinstance/pull/52>`__.


We use following Python excellence under the hood:

-  `Flex <https://github.com/pipermerriam/flex>`__, by Piper Merriam,
   for Swagger 2.0 validation
-  `GenSON <https://github.com/wolverdude/GenSON>`__, by Jon
   "wolverdude" Wolverton, for JSON Schema generator
-  `jsonpath-ng <https://github.com/h2non/jsonpath-ng>`__,
   by Tomas Aparicio and Kenneth Knowles, for handling JSONPath queries
-  `jsonschema <https://github.com/Julian/jsonschema>`__, by Julian
   Berman, for JSON Schema validator
-  `pygments <http://pygments.org>`__, by Georg Brandl et al.,
   for JSON syntax coloring, in terminal `Output`
-  `requests <https://github.com/requests/requests>`__, by Kenneth
   Reitz et al., for making HTTP requests

See `requirements.txt <https://github.com/asyrjasalo/RESTinstance/blob/master/requirements.txt>`__ for all the direct run time dependencies.

REST your mind, OSS got your back.
