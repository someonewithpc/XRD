*******
XRD
*******

PHP library to parse and generate
`Extensible Resource Descriptor (XRD) Version 1.0`__ files.
It supports loading and saving XML (XRD) and JSON (JRD) markup.

XRD and JRD files are used for ``.well-known/host-meta`` files as standardized
in `RFC 6415: Web Host Metadata`__.

Webfinger__, based on JRD, can be used to discover information about users
by just their e-mail address, e.g. their OpenID provider URL.

The XRD format supercedes the XRDS format defined in XRI 2.0, which is used in
the `Yadis communications protocol`__.

This is a modern PHP version of https://github.com/pear/XML_XRD

__ http://docs.oasis-open.org/xri/xrd/v1.0/xrd-1.0.html
__ http://tools.ietf.org/html/rfc6415
__ http://tools.ietf.org/html/draft-ietf-appsawg-webfinger-13
__ http://yadis.org/

.. contents::

========
Examples
========


Loading XRD files
=================

Load from file
--------------
::

    <?php
    use XRD\Document;
    use XRD\XRDException;

    $xrd = new Document;
    try {
        $xrd->loadFile('/path/to/my.xrd', 'xml');
    } catch (XRDException $e) {
        die('Loading XRD file failed: '  . $e->getMessage());
    }


Load from string
----------------
::

    <?php
    $myxrd = <<<XRD
    <?xml version="1.0"?>
    <XRD>
     ...
    XRD;

    use XRD\Document;
    $xrd = new Document;
    try {
        $xrd->loadString($myxrd, 'xml');
    } catch (XRDException $e) {
        die('Loading XRD string failed: '  . $e->getMessage());
    }


Verification
============

Verify subject
--------------
Check if the XRD file really describes the resource/URL that we requested the
XRD for::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    if (!$xrd->describes('http://example.org/')) {
        die('XRD document is not the correct one for http://example.org/');
    }

The ``<subject>`` and all ``<alias>`` tags are checked.



Link finding
============

Get all links
-------------
::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    foreach ($xrd as $link) {
        echo $link->rel . ': ' . $link->href . "\n";
    }


Get link by relation
--------------------
Returns the first link that has the given ``relation``::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    $idpLink = $xrd->get('lrdd');
    echo $idpLink->rel . ': ' . $idpLink->href . "\n";


Get link by relation + optional type
------------------------------------
If no link with the given ``type`` is found, the first link with the correct
``relation`` and an empty ``type`` will be returned::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    $link = $xrd->get('lrdd', 'application/xrd+xml');
    echo $link->rel . ': ' . $link->href . "\n";


Get link by relation + type
---------------------------
The ``relation`` and the ``type`` both need to match exactly::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    $link = $xrd->get('lrdd', 'application/xrd+xml', false);
    echo $link->rel . ': ' . $link->href . "\n";


Get all links by relation
-------------------------
::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    foreach ($xrd->getAll('lrdd') as $link) {
        echo $link->rel . ': ' . $link->href . "\n";
    }


Properties
==========

Get a single property
---------------------
::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    if (isset($xrd['http://spec.example.net/type/person'])) {
        echo $xrd['http://spec.example.net/type/person'] . "\n";
    }


Get all properties
------------------
::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    foreach ($xrd->getProperties() as $property) {
        echo $property->type . ': ' . $property->value . "\n";
    }


Get all properties of a type
----------------------------
::

    <?php
    use XRD\Document;
    $xrd = new Document;
    $xrd->loadFile('http://example.org/.well-known/host-meta');
    foreach ($xrd->getProperties('http://spec.example.net/type/person') as $property) {
        echo $property->type . ': ' . $property->value . "\n";
    }


Working with Links
==================

Accessing link attributes
-------------------------
::

    <?php
    $link = $xrd->get('http://specs.openid.net/auth/2.0/provider');

    $title = $link->getTitle('de');
    $url   = $link->href;
    $urlTemplate = $link->template;
    $mimetype    = $link->type;

Additional link properties
--------------------------
Works just like properties in the XRD document::

    <?php
    $link = $xrd->get('http://specs.openid.net/auth/2.0/provider');
    $prop = $link['foo'];


Generating XRD files
====================

.well-known/host-meta
---------------------
As described by RFC 6415::

    <?php
    use XRD\Document;
    use XRD\Element\Link;

    $xrd = new Document;
    $x->subject = 'example.org';
    $x->aliases[] = 'example.com';
    $x->links[] = new Link(
        'lrdd', 'http://example.org/gen-lrdd.php?a={uri}',
        'application/xrd+xml', true
    );
    echo $x->to('xml');
    ?>

If you want a JSON file for JRD::

    echo $x->to('json');


Webfinger file
--------------
::

    <?php
    use XRD\Document;
    use XRD\Element\Link;

    $xrd = new Document();
    $x->subject = 'user@example.org';
    
    //add link to the user's OpenID
    $x->links[] = new Link(
        'http://specs.openid.net/auth/2.0/provider',
        'http://id.example.org/user'
    );
    //add link to user's home page
    $x->links[] = new Link(
        'http://xmlns.com/foaf/0.1/homepage',
        'http://example.org/~user/'
    );
    
    echo $x->to('jrd');
    ?>



==============
Error handling
==============

When loading a file, exceptions of type ``XML_XRD_Exception`` may be thrown.
All other parts of the code do not throw exceptions but fail gracefully by returning
``null``, e.g. when a property does not exist.

Using ``loadFile()`` may result in PHP warnings like::

  Warning: simplexml_load_file(https://example.org/) failed to open stream: Connection refused

This cannot be prevented properly, so you either have to silence it with ``@``
or fetch the file manually and use ``loadString()``.
    

====
TODO
====

- XML signature verification
- (very optional) XRDS (multiple XRD)?

=====
Links
=====

- `XRD 1.0 standard specification`__
- `OASIS XRI committee`__
- `WebFinger protocol draft`__
- `WebFinger: Common Link relations`__
- `More link relations`__
- `RFC 5785: Defining Well-Known Uniform Resource Identifiers`__
- `RFC 6415: Web Host Metadata`__
- `WebFinger draft`__

__ http://docs.oasis-open.org/xri/xrd/v1.0/xrd-1.0.html
__ http://www.oasis-open.org/committees/tc_home.php?wg_abbrev=xri
__ http://code.google.com/p/webfinger/wiki/WebFingerProtocol
__ http://code.google.com/p/webfinger/wiki/CommonLinkRelations
__ http://search.cpan.org/~tobyink/WWW-Finger-0.101/lib/WWW/Finger/Webfinger.pm
__ http://tools.ietf.org/html/rfc5785
__ http://tools.ietf.org/html/rfc6415
__ http://tools.ietf.org/html/draft-ietf-appsawg-webfinger-13
