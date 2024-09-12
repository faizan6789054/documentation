==================
Write Data Modules
==================

.. important::
   This tutorial assumes familiarity with the :doc:`server_framework_101` tutorial and the
   :doc:`define_module_data` tutorial.

Although as developpers, we prefer to have the full power of Python to write our modules,
it is sometimes not possible to do so; typically on managed hosting solutions which do not
allow the deployment of custom Python code like the `Odoo.com <https://www.odoo.com/start>`_
platform.

However, the flexible nature of Odoo is meant to allow customizations out of the box. Whilst
a lot is possible with :doc:`Studio </applications/studio>`, it is also possible to define
models, fields and logic in :doc:`XLM Data File <define_module_data>`. This makes it easier
to develop, maintain and deploy these customizations.

In this tutorial, we will learn how to define models, fields and logic in XML data files.
We will also see the limitations of this approach to module development.

Problem Statement
=================

Like in the :doc:`server_framework_101` tutorial, we will be working on Real Estate concepts.

Our goal is to create a new application to manage Real Estate properties in a similar (albeit
simpler) way to the :doc:`server_framework_101` tutorial. We will define the models, fields and
logic in XML data files instead of Python files.

At the end of this tutorial, we will be able to achieve the following in our app:

- Manage Real Estate properties that are for sale
- Publish these properties on a website
- Accept offers online from the website
- Invoice the buyer when the property is sold

Module Structure
================

Like in any developmnet project, a clear structure makes it easier to manage and maintain the code.

Whilst in normal Odoo modules, you will have a mix of Python and XML files, in this case, we will
only have XML files. Therefore, it is expected that your work tree will look something like this:

.. code-block:: bash

  estate
  ├── actions
  │   └── *.xml
  ├── models
  │   └── *.xml
  ├── security
  │   └── ir.model.access.csv
  ├── views
  │   └── *.xml
  ├── __init__.py
  └── __manifest__.py

The only Python files you will have are the ``__init__.py`` and ``__manifest__.py`` files. The
``__manifest__.py`` file will be the same as for any Odoo module, but will also import its models
in the `data` list.

Don't forget that the files in the `data` list in the ``__manifest__.py`` file will be loaded in
the order they are listed, so you will typically start by loading the model files first (as most
other files will depend on them).

The ``__init__.py`` file is empty, but is required for Odoo to recognize the module.

Deploying the Module
====================

To deploy the module, you will need to create a zip file of the module and upload it to your
Odoo instance. Make sure that the module `base_import-module` is installed on your instance,
then go to the :menuselection:`Apps --> Import Module` and upload the zip file. You must be
in :ref:`developer mode <developer-mode>` to see the `Import Module` menu item.

If you modify the module, you will need to create a new zip file and upload it again, which
will reload all the data in the module. Note however that some operations are not possible,
like changing the type of a field you created previously. Note also that data that was created
by a previous version of the module will not be deleted (e.g. a field you decide to remove, etc.).
In general, the simplest way to handle this is to start with a fresh database or to uninstall
the module prior to uploading the new version.

When uploading a module, the wizard will accept 2 options:

- `Force init`: if your module is already installed and you upload it again, checking this
  option will force the update of all data marked as `noupdate="1"` in the XML files.
- `Import demo data`: self explanatory

It is also possible to deploy the module using the `odoo-bin` command line tool with the `deploy`
command:

.. code-block:: bash

  $ odoo-bin deploy <path_to_your_module> https://<your_odoo_instance> --login <your_login> --password <your_password>

This command also accepts the `--force` option, which is equivalent to the `Force init` option
in the wizard.

Note that the user you use to deploy the module must have `Administration/Settings` access rights.


Models and Basic Fields
=======================

As you can imagine, defining models and fields in XML files is not as straightforward as in Python.

Since data files are read sequentially, you must define the elements in the right order.
For example, you must define a model before you can define a field on that model, and you
must define fields before adding them to a view.

In addition, XML is simply much more verbose than Python.

Let's start by defining a simple model to represent a Real Estate property in the `models`
directory of our module.

Odoo models are stored in database as `ir.model` records. Like any other record, they can be
defined in XML files:

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <record model="ir.model" id="model_real_estate_property">
            <field name="name">Real Estate Property</field>
            <field name="model">x_estate.property</field>
        </record>
    </odoo>

Note that all models and fields defined in data files must be prefixed with `x_`; this is
mandatory and is used to differentiate them from models and fields defined in Python files.

By default, Odoo will automatically add several fields to the model:

- `id`: the primary key of the record
- `create_date`: the date the record was created
- `write_date`: the date the record was last updated
- `create_uid`: the user that created the record
- `write_uid`: the user that last updated the record
- `x_name`: a char field that is used to display the record in the interface

We can also add several fields to our new model. Let's add some simple fields, like a selling
price (float), a description (as html), and a city name (as a char).

Like for models, fields are simply records of the `ir.model.fields` model and can be
defined as such in data files:

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <odoo>
        <!-- ...model definition from before... -->
        <record model="ir.model.fields" id="field_real_estate_property_selling_price">
            <field name="model_id" ref="estate.model_real_estate_property" />
            <field name="name">x_selling_price</field>
            <field name="field_description">Selling Price</field>
            <field name="ttype">float</field>
            <field name="required">True</field>
        </record>
        <record model="ir.model.fields" id="field_real_estate_property_description">
            <field name="model_id" ref="estate.model_real_estate_property" />
            <field name="name">x_description</field>
            <field name="field_description">Description</field>
            <field name="ttype">htlm</field>
        </record>
        <record model="ir.model.fields" id="field_real_estate_property_city">
            <field name="model_id" ref="estate.model_real_estate_property" />
            <field name="name">x_city</field>
            <field name="field_description">City</field>
            <field name="ttype">char</field>
        </record>
    </odoo>

Quite a few attributes of your new field can be set this way; for basic fields,
the basic attributes are:

- `name`: the technical name of the field (must begin with `x_`)
- `field_description`: the label of the field
- `help`: a help text for the field, displayed in the interface
- `ttype`: the type of the field (e.g. `char`, `integer`, `float`, `html`, etc.)
- `required`: whether the field is required or not (default: `False`)
- `readonly`: whether the field is read-only or not (default: `False`)
- `index`: whether the field is indexed or not (default: `False`)
- `copied`: whether the field is copied when duplicating a record or not (default: `True`
  for scalar non-computed fields)
- `translate`: whether the field is translatable or not (default: `False`)

Attributes are also available to control HTML sanitization as well as other, more advanced
features; for a complete list, refer to the `ir.model.fields` model in the database available
in the :menuselection:`Settings --> Technical --> Database Structure --> Fields` menu or
see the `ir.model.fields` model definition in the `base` module.

Necessity is the mother of invention
====================================

Some developments seem impossible to achieve with data files, but are not. This section
contains several examples of developments that can be achieved with data files, but are
not always obvious or well-documented.

Website controllers
-------------------

HTTP Controllers in Odoo are usually defined in the `controllers` directory of a module;
in data modules, it is possible to define server actions that behave as controllers if
the website module is installed.

When the website module is installed, server actions can be marked as `Available on the website`
and given a path (the full path is always prefixed with /actions to avoid URL collisions);
the global `request` object is made available in the local variables of the code server action.

Since it is not possible to return a value from within a server action, to define the response
to return, one can assign a response-like object to the `response` variable, which will be
returned to the website automatically. The `request` object provides several methods to
access the body of the request:

- `request.get_http_params()`: extract key-value pairs from the query string and the forms
  present in the body (both application/x-www-form-urlencoded and multipart/form-data).
- `request.get_json_data()`: extract the JSON data from the body of the request.


Here is an example of a simple website controller that will return a list of properties
when the URL `/actions/estate` is called:

.. code-block:: xml

    <record model="ir.actions.server" id="server_action_estate_list">
        <field name="name">Estate List Controller</field>
        <field name="model_id" ref="x_estate.model_real_estate_property" />
        <field name="website_published">True</field>
        <field name="website_path">estate</field>
        <field name="code">
            response.html = '<html><body><h1>Properties</h1><ul>'
            for property in request.env['x_estate.property'].search([]):
                response.html += f'<li>{property.x_name}</li>'
            response.html += '</ul></body></html>'
        </field>
    </record>

Several useful methods are available in the `request` object to facilitate the generation of the
response object:

- `request.render(template, qcontext=None, lazy=True, **kw)` to render a QWeb template using its
  xmlid; the extra keyword arguments are forwarded to the `werkzeug.Response` object (e.g. to set
  cookies, headers, etc.)
- `request.redirect(location, code=303, local=True)` to redirect to a different URL; the `local`
  argument is used to specify whether the redirection should be relative to the website or not
  (default: `True`).
- `request.notfound()` to return a `werkzeug.HTTPException` exception to signal a 404 error to
  the website.
- `request.make_response(data, headers=None, cookies=None, status=200)` to manually create a
  `werkzeug.Response` object; the `status` argument is the HTTP status code to return (default:
  200).
- `request.make_json_response(data, headers=None, cookies=None, status=200)` to manually create a
  JSON response; the data will be json-serialiazed using `json.dumps` utility; this can be useful
  to set up machine-to-machine communications via API calls.


For implementation details or other (less common) methods, refer to the `Request` object's
implementation in the `odoo.http` module.

Note that security concerns are left to the developer (typically through security rules or
by using `sudo` to access records).

.. note::

  The model used in the `model_id` field of the server action must be acessible to the public
  user for the read operation for this server action to run; otherwise the server action will
  return a 403 error.

.. exercise::

  Add a JSON API to your module so that external services can retrieve a list of properties
  for sale.

  - add a new `x_api_published` field to the model to control whether the properties are
    published on the API or not
  - add an access right record to allow public users to read the model
  - add a record rule so that only properties marked as `x_api_published` are returned by
    the API for the public user
  - add a server action to return a list of properties in JSON format when the URL
    `/actions/api/estate` is called


Overriding Python Models
------------------------

Via UI Elements
~~~~~~~~~~~~~~~

Unlike in Python modules, it is not possible to override a Python model's method cleanly.

However, it is possible (in some cases) to replace the elements of the UI that call
these methods and to intercept the calls to these methods in a server action.


Via Automation Rules
~~~~~~~~~~~~~~~~~~~~

It is possible to replace a Python model's method via an automation rule, but this is
more complex and less powerful than doing it in data files.
