=========
vmod_bofh
=========

-------------------
Varnish BOFH Module
-------------------

SYNOPSIS
========

::

        import bofh;


DESCRIPTION
===========

Varnish vmod that picks a random entry from a list of builtin BOFH excuses
each time the function is called. For use with synthetic error pages.

BOFH 101: Intended to confuse users, make them feel responsible, or blame
anything under the sun but us, when backend services go AWOL. This should
dissuade those pesky users from storming NOC or customer support with their
silly and righteous demands about uptime and availability.

Beware: The use of this strategy in company production environments may put
ones' reputation as a reliable Sysadmin at risk. As such, after deployment
one should preemptively prepare to upgrade ones' BOFH arsenal against inside
threats of forced unemployment.

The list of excuses is hardcoded for fire-and-forget deployments, so no time
is wasted on unimportant details like configuration setups. Who has time to
waste on such irrelevant things anyway.

(Would you be looking for a Sysadmin, per chance?)


FUNCTIONS
=========

excuse
------

Prototype::

        excuse()
Return value::

	STRING

Description:

        Function that returns a random excuse.

Example::

        sub vcl_synth {
                set resp.reason = bofh.excuse();
                // return (deliver);
        }
        sub vcl_backend_error {
                set beresp.reason = bofh.excuse();
                // return (deliver);
        }


fix_excuse
----------

Prototype::

        fix_excuse(INT)

Return value::

        VOID

Description:

        The cheat mode. Calling this procedure with a list-index number will fix the
        return value for the next call of `excuse()` to this specific excuse number.

Example::

        bofh.fix_excuse(390)
        set resp.reason = bofh.excuse();
        //set resp.status = 777;


INSTALLATION
============

The source tree is based on autotools to configure the building, and
does also have the necessary bits in place to do functional unit tests
using the ``varnishtest`` tool.

Building requires the Varnish header files and uses pkg-config to find
the necessary paths.

Usage::

 ./autogen.sh
 ./configure

If you have installed Varnish to a non-standard directory, call
``autogen.sh`` and ``configure`` with ``PKG_CONFIG_PATH`` pointing to
the appropriate path. For instance, when varnishd configure was called
with ``--prefix=$PREFIX``, use

 PKG_CONFIG_PATH=${PREFIX}/lib/pkgconfig
 export PKG_CONFIG_PATH

Make targets:

* ``make`` - builds the vmod.
* ``make install`` - installs your vmod.
* ``make check`` - runs the unit tests in ``src/tests/*.vtc``
* ``make distcheck`` - run check and prepare a tarball of the vmod.

Build Debian package (does all of the above as well):

* ``./autogen.sh``
* ``dpkg-buildpackage -rfakeroot -us -uc -I".git*"``
* ``dpkg -I ../libvmod-bofh_*.deb`` - verify built package
* ``dpkg -c ../libvmod-bofh_*.deb`` - and contents
* ``sudo dpkg -i ../libvmod-bofh_*.deb`` - install

Installation directories
------------------------

By default, the vmod ``configure`` script installs the built vmod in
the same directory as Varnish, determined via ``pkg-config(1)``. The
vmod installation directory can be overridden by passing the
``VMOD_DIR`` variable to ``configure``.

Other files like man-pages and documentation are installed in the
locations determined by ``configure``, which inherits its default
``--prefix`` setting from Varnish.


COMMON PROBLEMS
===============

* configure: error: Need varnish.m4 -- see README.rst

  Check if ``PKG_CONFIG_PATH`` has been set correctly before calling
  ``autogen.sh`` and ``configure``

* Incompatibilities with different Varnish Cache versions

  Make sure you build this vmod against its correspondent Varnish Cache version.

  It has been compiled against and tested with Varnish 4.1, and should probably
  build and work with 4.0 as well.
  Well, there was no reason to actually test it (god forbid), ... but it
  _might_ work!
  Check the vmod-example source code if you need to adapt it for other versions
  (which has been used as a template for this).


USAGE EXAMPLE
=============

In your VCL you could then use this vmod along the following lines::

        import bofh;

        sub vcl_deliver {
                // Nobody cares about the response reason on 200 anyway. OK???
                return (synth(200, bofh.excuse()));
        }

Then check the results

.. image:: https://cloud.githubusercontent.com/assets/438293/13832805/975b7560-ebd7-11e5-8eae-61feee712e05.gif
   :alt: 200!!! OK???

in your favorite web-debugging tool

.. image:: https://cloud.githubusercontent.com/assets/438293/13832809/a2df3412-ebd7-11e5-8e9f-65a73c19f1ea.gif
   :alt: 404


And a more elaborate example::

        import bofh;

        sub vcl_deliver {
                # Return some excuse numbered 389
                return (synth(490, "BOFH"));
        }

        sub vcl_synth {
                if (resp.reason == "BOFH") {
                        # Get the excuse number

                        # `resp.status` only accepts valid status numbers (>=100?),
                        # so we just work around that by subtracting a count of 100
                        # (0 is valid, negative numbers are not).
                        bofh.fix_excuse(resp.status - 100);

                        set resp.reason = bofh.excuse();

                        # reset to "real" response status
                        set resp.status = 404;

                        // let it go on to the default vcl_synth error page
                }
        }

Which should result in the following response:

.. image:: https://cloud.githubusercontent.com/assets/438293/13834828/fa5f4d32-ebe9-11e5-9d46-c537ae7145c3.gif
   :alt: Cache Miss


ABOUT
=====

"nyov", 2016

Released into the Public Domain.


Originally written in 2008 for Varnish 1.x as a few lines of VCL inline-c,
using ``VRT_synth_page()``. It was compiled into a VMOD as a learning experience.

A sample VCL with synthetic error page is included. If you were truly (un)lucky,
you might have seen that one in action between 2008 and 2011 on some local german
newsoutlet (but don't go blab about it then and make me look bad):

.. image:: https://cloud.githubusercontent.com/assets/438293/13832844/f10ff9d2-ebd7-11e5-9fc1-f74fe640c37b.gif
   :alt: Original look
