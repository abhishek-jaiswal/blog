---
layout: post
title:  "ODOO V8 API: Everything about Decorator and Metaclass"
date:   2014-11-15 20:10:10
categories: odoov8
tags: odoo-v8
image: /blog/assets/article_images/everything.jpg
---
<section class="container content">
   <div class="title">
        <h1>ODOO V8 API: Everything about Decorator and Metaclass</h1>
        <div class="when">2014-11-15 20:10:10</div>
    </div>

<div id='content'></div>

# API
  An API(Application Programming Interface) is a set of defined functions and
  procedures that allow the creation of applications which access the features
  or data of an operating system, application, or other service.
  (Source [Google](#https://www.google.co.in/search?q=api+defination&oq=api+defination))

Metaclass
==========================
A metaclass is defined as "the class of a class". Any class whose instances
are themselves classes, is a metaclass.
(Source [wikipedia](#http://en.wikipedia.org/wiki/Metaclass))

#### Some Explanations what we can do with metaclass
* Enforce different inheritance semantics, e.g. automatically call base
  class methods when a derived class overrides.
* Implement class methods (e.g. if the first argument is not named 'self')
  for precondition and post-condition checking.
* Implement that each instance is initialized with copies of all class variables.
* Implement a different way to store instance variables (e.g. in a list
  kept outside the the instance but indexed by the instance's id()).
* Automatically wrap or trap all or certain methods:
* Used for tracing
* Used for precondition and post-condition checking
* Used for synchronized methods
* Used for automatic value caching

####Metaclass's __new__ and __init__:
To control the creation and initialization of the class in the metaclass,
you can implement the metaclass's __new__ method and/or __init__ constructor.
Most real-life metaclasses will probably override just one of them. __new__
should be implemented when you want to control the creation of a new object
(class in our case), and __init__ should be implemented when you want to
control the initialization of the new object after it has been created.

#### Features of New API
Api is bridge filling the gap between old "traditional" and new "record" style

In old traditional style , Parameter like database cursor, user id,
context dictionary, and record are manually passed in the function in the form
of parameters such as (cr, uid, ids, context)
Example

{% highlight python %}
model = self.pool.get(MODEL)
ids = model.search(cr, uid, DOMAIN, context=context)
for rec in model.browse(cr, uid, ids, context=context):
print rec.name
model.write(cr, uid, ids, VALUES, context=context)
{% endhighlight %}

In new API style those are hidden into model instances, which gives it more
pythonic and object-oriented feel. which can be access as (self._cr, self._uid ..)

example above(old style)can be written as:

{% highlight python %}
recs = self.env[MODEL] # retrieve an instance of MODEL
recs = recs.search(DOMAIN) # search returns a recordset
for rec in recs: # iterate over the records
print rec.name
recs.write(VALUES) # update all records in recs
{% endhighlight %}

Methods written in the "traditional" style are automatically decorated,
following some heuristics based on parameter names.
#### Meta classes
__class Meta(type):__
    It is used to automatically decorates traditional-style methods by guessing
    their API. It also implements the inheritance of the :func:'returns'
    decorators.



__Metaclass's __new__:__
    It is used to automatically decorates traditional-style methods by
    guessing their API.

#### Decorator's
A decorator is just a callable that takes a function as an argument and
returns a replacement function

__The Decorator's which is used to decorate the traditional-style method:__

`@api.cr:`
Decorate a traditional-style method that takes 'cr' as a parameter.
Such a method may be called in both record and traditional styles,
like:

`@api.cr_context:`
Decorate a traditional-style method that takes 'cr', 'context' as
parameters.

`@api.cr_uid:`
Decorate a traditional-style method that takes 'cr', 'uid' as
parameters.

`@api.cr_uid_context:`
Decorate a traditional-style method that takes 'cr', 'uid', 'context' as
parameters.
Such a method may be called in both record and traditional
styles, like:

recs.method(args)
model.method(cr, uid, args, context=context)

`@api.cr_uid_ids:`
Decorate a traditional-style method that takes 'cr', 'uid', 'ids'
as parameters. Such a method may be called in both record and
traditional styles. In the record style, the method automatically
loops on records.

`@api.cr_uid_id_context:`
Decorate a traditional-style method that takes 'cr', 'uid', 'id',
'context' as parameters. Such a method:

`@api.cr_uid_ids_context:`
Decorate a traditional-style method that takes 'cr', 'uid', 'ids',
'context' as parameters. Such a method:


__The Decorator's which is used to decorate the new record-style method:__

`@api.model:`
Decorate a record-style method where 'self' is a recordset, but its contents is not relevant, only the model is. Such a method:

    @api.model
    def method(self, args):
        ...

may be called in both record and traditional styles, like:

    # recs = model.browse(cr, uid, ids, context)
    recs.method(args)

    model.method(cr, uid, args, context=context)

Notice that no 'ids' are passed to the method in the traditional style.

`@api.one:`
Decorate a record-style method where 'self' is expected to be a singleton instance. The decorated method automatically loops on records, and makes a list with the results. In case the method is decorated with @returns, it concatenates the resulting instances. Such a method:

    @api.one
    def method(self, args):
        return self.name

may be called in both record and traditional styles, like::

    # recs = model.browse(cr, uid, ids, context)
    names = recs.method(args)

    names = model.method(cr, uid, ids, args, context=context)

Each time 'self' is redefined as current record.

`@api.multi:`
Decorate a record-style method where 'self' is a recordset. The method typically defines an operation on records. Such a method:

    @api.multi
    def method(self, args):
        ...

may be called in both record and traditional styles, like::

    # recs = model.browse(cr, uid, ids, context)
    recs.method(args)

    model.method(cr, uid, ids, args, context=context)

`@api.constrains:`
Decorates a constraint checker. Each argument must be a field name used in the check:

    @api.one
    @api.constrains('name', 'description')
    def _check_description(self):
        if self.name == self.description:
            raise ValidationError("Fields name and description must be different")

Invoked on the records on which one of the named fields has been modified.

Should raise :class:'~openerp.exceptions.ValidationError' if the validation failed.

`@api.onchange:`
Return a decorator to decorate an onchange method for given fields. Each argument must be a field name:

    @api.onchange('partner_id')
    def _onchange_partner(self):
        self.message = "Dear %s" % (self.partner_id.name or "")

In the form views where the field appears, the method will be called when one of the given fields is modified. The method is invoked on a pseudo-record that contains the values present in the form. Field assignments on that record are automatically sent back to the client.

`@api.depends:`
Return a decorator that specifies the field dependencies of a "compute" method (for new-style function fields). Each argument must be a string that consists in a dot-separated sequence of field names:

    pname = fields.Char(compute='_compute_pname')

    @api.one
    @api.depends('partner_id.name', 'partner_id.is_company')
    def _compute_pname(self):
        if self.partner_id.is_company:
            self.pname = (self.partner_id.name or "").upper()
        else:
            self.pname = self.partner_id.name

One may also pass a single function as argument. In that case, the dependencies are given by calling the function with the field's model.

`@api.returns:`
Return a decorator for methods that return instances of 'model'.

    :param model: a model name, or 'self' for the current model

    :param downgrade: a function 'downgrade(value)' to convert the
        record-style 'value' to a traditional-style output

The decorator adapts the method output to the api style: 'id', 'ids' or 'False' for the traditional style, and recordset for the record style:

    @model
    @returns('res.partner')
    def find_partner(self, arg):
        ...     # return some record

    # output depends on call style: traditional vs record style
    partner_id = model.find_partner(cr, uid, arg, context=context)

    # recs = model.browse(cr, uid, ids, context)
    partner_record = recs.find_partner(arg)

Note that the decorated method must satisfy that convention.

Those decorators are automatically *inherited*: a method that overrides a decorated existing method will be decorated with the same
'@returns(model)'.

The Decorator's which is used to decorate a method that supports the old-style API only:

1. @api.v7:

Decorate a method that supports the old-style api only. A new-style api may be provided by redefining a method with the same name and decorated with :func:'~.v8':

    @api.v7
    def foo(self, cr, uid, ids, context=None):
        ...

    @api.v8
    def foo(self):
        ...

Note that the wrapper method uses the docstring of the first method.

The Decorator's which is used to decorate a method that supports the new-style API only:

1. @api.v8:

Decorate a method that supports the new-style api only. An old-style api may be provided by redefining a method with the same name and decorated with :func:'~.v7':

    @api.v8
    def foo(self):
        ...

    @api.v7
    def foo(self, cr, uid, ids, context=None):
        ...

Note that the wrapper method uses the docstring of the first method.
