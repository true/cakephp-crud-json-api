Setup
=====

Before you can start producing JSON API you will have to set up
your application by following the steps in this section.

Controller
^^^^^^^^^^

Attach the listener using the components array if you want to attach
it to all controllers, application wide, and make sure ``RequestHandler``
is loaded **before** ``Crud``.

.. code-block:: php

  <?php
  class AppController extends Controller {

    public function initialize()
    {
      $this->loadComponent('RequestHandler');
      $this->loadComponent('Crud.Crud', [
        'actions' => [
          'Crud.Index',
          'Crud.View'
        ],
        'listeners' => ['CrudJsonApi.JsonApi']
      ]);
    }

Alternatively, attach the listener to your controllers ``beforeFilter``
if you prefer attaching the listener to only specific controllers on the fly.

.. code-block:: php

  <?php
  class SamplesController extends AppController {

    public function beforeFilter(\Cake\Event\Event $event) {
      parent::beforeFilter();
      $this->Crud->addListener('CrudJsonApi.JsonApi');
    }
  }

Exception Handler
^^^^^^^^^^^^^^^^^

The JsonApi listener overrides the ``Exception.renderer`` for ``jsonapi`` requests,
so in case of an error, a standardized error will be returned,
according to the JSON API specification.

Create a custom exception renderer by extending the Crud's ``JsonApiExceptionRenderer``
class and enabling it with the ``exceptionRenderer`` configuration option.

.. code-block:: php

  <?php
  class AppController extends Controller {

    public function initialize()
    {
      parent::initialize();
      $this->Crud->config(['listeners.jsonapi.exceptionRenderer' => 'App\Error\JsonApiExceptionRenderer']);
    }
  }

**Note:** However if you are using CakePHP 3.3+'s PSR7 middleware feature the ``exceptionRenderer``
config won't be used and instead you will have to set the ``Error.exceptionRenderer``
config in ``config/app.php`` to ``'CrudJsonApi\Error\JsonApiExceptionRenderer'`` as following:

.. code-block:: php

    'Error' => [
        'errorLevel' => E_ALL,
        'exceptionRenderer' => 'CrudJsonApi\Error\JsonApiExceptionRenderer',
        'skipLog' => [],
        'log' => true,
        'trace' => true,
    ],

Routing
^^^^^^^

Only controllers explicitly mapped can be exposed as API resources so make sure
to configure your global routing scope in ``config/routes.php`` similar to:

.. code-block:: phpinline

  const API_RESOURCES = [
    'Countries',
    'Currencies'
  ];

  Router::scope('/', function ($routes) {
    foreach (API_RESOURCES as $apiResource) {
        $routes->resources($apiResource, [
            'inflect' => 'dasherize'
        ]);
    }
  });

Request detector
^^^^^^^^^^^^^^^^

The JsonApi Listener adds the ``jsonapi`` request detector
to your ``Request`` object which checks if the request
contains a ``HTTP Accept`` header set to ``application/vnd.api+json``
and can be used like this inside your application:

.. code-block:: php

  if ($this->request->is('jsonapi')) {
    return('cool, using JSON API');
  }

.. note::

To make sure the listener won't get in your way it will
return ``null`` for all requests unless ``is('jsonapi')`` is true.
