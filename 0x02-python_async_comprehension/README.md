<img src="https://s3.amazonaws.com/alx-intranet.hbtn.io/uploads/medias/2019/12/ee85b9f67c384e29525b.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&amp;X-Amz-Credential=AKIARDDGGGOUSBVO6H7D%2F20240323%2Fus-east-1%2Fs3%2Faws4_request&amp;X-Amz-Date=20240323T014525Z&amp;X-Amz-Expires=86400&amp;X-Amz-SignedHeaders=host&amp;X-Amz-Signature=45482f1bbd9752454f7db81429be0c9db1bcb26813b2b45cacb04814c77f016b" alt="" loading="lazy" style="">
PEP: 530
Title: Asynchronous Comprehensions
Version: $Revision$
Last-Modified: $Date$
Author: Yury Selivanov <yury@edgedb.com>
Discussions-To: python-dev@python.org
Status: Final
Type: Standards Track
Content-Type: text/x-rst
Created: 03-Sep-2016
Python-Version: 3.6
Post-History: 03-Sep-2016


Abstract
========

:pep:`492` and :pep:`525` introduce support for native coroutines and
asynchronous generators using ``async`` / ``await`` syntax.  This PEP
proposes to add asynchronous versions of list, set, dict comprehensions
and generator expressions.


Rationale and Goals
===================

Python has extensive support for synchronous comprehensions, allowing
to produce lists, dicts, and sets with a simple and concise syntax.  We
propose implementing similar syntactic constructions for the
asynchronous code.

To illustrate the readability improvement, consider the following
example::

    result = []
    async for i in aiter():
        if i % 2:
            result.append(i)

With the proposed asynchronous comprehensions syntax, the above code
becomes as short as::

    result = [i async for i in aiter() if i % 2]

The PEP also makes it possible to use the ``await`` expressions in
all kinds of comprehensions::

    result = [await fun() for fun in funcs]


Specification
=============

Asynchronous Comprehensions
---------------------------

We propose to allow using ``async for`` inside list, set and dict
comprehensions.  Pending :pep:`525` approval, we can also allow creation
of asynchronous generator expressions.

Examples:

* set comprehension: ``{i async for i in agen()}``;

* list comprehension: ``[i async for i in agen()]``;

* dict comprehension: ``{i: i ** 2 async for i in agen()}``;

* generator expression: ``(i ** 2 async for i in agen())``.

It is allowed to use ``async for`` along with  ``if`` and ``for``
clauses in asynchronous comprehensions and generator expressions::

    dataset = {data for line in aiter()
                    async for data in line
                    if check(data)}

Asynchronous comprehensions are only allowed inside an ``async def``
function.

In principle, asynchronous generator expressions are allowed in
any context.  However, in Python 3.6, due to ``async`` and ``await``
soft-keyword status, asynchronous generator expressions are only
allowed in an ``async def`` function.  Once ``async`` and ``await``
become reserved keywords in Python 3.7, this restriction will be
removed.


``await`` in Comprehensions
---------------------------

We propose to allow the use of ``await`` expressions in both
asynchronous and synchronous comprehensions::

    result = [await fun() for fun in funcs]
    result = {await fun() for fun in funcs}
    result = {fun: await fun() for fun in funcs}

    result = [await fun() for fun in funcs if await smth]
    result = {await fun() for fun in funcs if await smth}
    result = {fun: await fun() for fun in funcs if await smth}

    result = [await fun() async for fun in funcs]
    result = {await fun() async for fun in funcs}
    result = {fun: await fun() async for fun in funcs}

    result = [await fun() async for fun in funcs if await smth]
    result = {await fun() async for fun in funcs if await smth}
    result = {fun: await fun() async for fun in funcs if await smth}

This is only valid in ``async def`` function body.


Grammar Updates
---------------

The proposal requires one change on the grammar level: adding the
optional "async" keyword to ``comp_for``::

    comp_for: [ASYNC] 'for' exprlist 'in' or_test [comp_iter]

The ``comprehension`` AST node will have the new ``is_async`` argument.


Backwards Compatibility
-----------------------

The proposal is fully backwards compatible.


Acceptance
==========

:pep:`530` was accepted by Guido, September 6, 2016 [1]_.


Implementation
==============

The implementation is tracked in issue 28008 [3]_.  The reference
implementation git repository is available at [2]_.


References
==========

.. [1] https://mail.python.org/pipermail/python-ideas/2016-September/042141.html

.. [2] https://github.com/1st1/cpython/tree/asyncomp

.. [3] http://bugs.python.org/issue28008


Acknowledgments
===============

I thank Guido van Rossum, Victor Stinner and Elvis Pranskevichus
for their feedback, code reviews, and discussions around this
PEP.

Copyright
=========

This document has been placed in the public domain.

..
   Local Variables:
   mode: indented-text
   indent-tabs-mode: nil
   sentence-end-double-space: t
   fill-column: 70
   coding: utf-8
   End:
