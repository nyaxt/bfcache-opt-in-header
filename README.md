# Explainer: Salvageable Opt-in Header

## Authors
* [@nyaxt](https://github.com/nyaxt)
* [@yoavweiss](https://github.com/yoavweiss)


## Participate

* [github.com/nyaxt/bfcache-opt-in-header/issues](https://github.com/nyaxt/bfcache-opt-in-header/issues)


## Introduction

Browsers today use an optimization feature when users navigate their browser’s history, called Back-Forward Cache (or BFCache for short). That optimization feature enables instant loading experience when users go back to a page they already recently visited.

However, that optimization cannot always be applied and different browsers have different heuristics that opt their users out of BFCache when encountered on the page when certain features are used by the web page.

Those heuristics can be overly cautious and there are cases where sites can properly cope with BFCache loads even when using features that the browser may consider “not BFCache safe”.

Our long-term desire is to make all pages BFCacheable and specify the interaction between BFCache and all web platform APIs in the long term, but in there are potential web compat implications due to pages not accounting for bfcache interaction with particular features (e.g. pages listening to unload rather than pagehide), so this transition will have to be gradual.

To facilitate this transition, we are proposing a new HTTP response header to allow websites opt-in into marking pages as “salvageable” (the HTML spec concept for BFCache-eligibility) in the presence of a particular feature in advance of us making this behaviour default.

Example:
```
BFCache-Opt-In: unload
```

For example, a web site can present the above in its response header to indicate that it wishes to remain BFCache eligible even if the page has an unload handler, which would not fire on BFCache navigation.


## Goals

Website can opt-in to claim itself to stay “salvageable” (BFCache eligible), while using features which behave differently or might not work well for BFCache, which can cause some user agents to not BFCache the page using them.

We would like to start from the “unload” handler, but we plan expand the header to cover the other features in the following list:

*   Unload handler
*   WebSocket
*   Inflight network requests
*   Cache-control: no-store


## Non-goals

The ``BFCache-Opt-In`` header is not required for the page to be BFCached. Many web page will not send the ``BFCache-Opt-In`` response header, and the page is still subject to BFCache. We even would like to extend automatic BFCache eligibility to the pages using the “disruptive” features enumerated in the Goals section, so eventually the header would not be needed anymore.

We don’t plan to use the header as a BFCache opt-out mechanism. Web sites who don’t wish to be BFCached should use `Cache-control: no-store` in the short term, or use the opt-out API currently being proposed at [html#5744](https://github.com/whatwg/html/issues/5744).

# The monkey patch to HTML spec

## “Unloading documents” patch

In [Unloading documents](https://html.spec.whatwg.org/multipage/browsing-the-web.html#unloading-documents), change the first sentence to the following: (change in **bold**)

A <code>[Document](https://html.spec.whatwg.org/multipage/dom.html#document)</code> has a _salvageable state_, which must initially be true, <strong>an _unload salvageable opt-in state_, which must be initialized to false</strong>, and a page showing flag, which must initially be false.

## “create and initialize a Document object” patch

Modify the [#initialise-the-document-object](https://html.spec.whatwg.org/multipage/browsing-the-web.html#initialise-the-document-object) accordingly.

Insert the following steps between the steps 13 and 14:

14. If navigationParams's [response](https://html.spec.whatwg.org/multipage/browsing-the-web.html#navigation-params-response) has a ``BFCache-Opt-In`` header, then:
    1. Let bfcacheOptInHeaderList be the result of [getting a structured field value](https://fetch.spec.whatwg.org/#concept-header-list-get-structured-header) given ``BFCache-Opt-In`` and "`list`" from the response's [header list](https://fetch.spec.whatwg.org/#concept-response-header-list).
    2. For each item `item` in bfcacheOptInHeaderList:
        1. If `item` is a token “unload”, set the document's _unload salvageable opt-in state_ to true.


## “Unload a document” patch

Modify the [#unload-a-document](https://html.spec.whatwg.org/multipage/browsing-the-web.html#unload-a-document) accordingly.

Insert the following steps between the steps 4 and 5.

4. A user agent can decide to avoid keeping a document alive in a session history entry (such that is can be reused later on history traversal) for any of the following reasons:
    1. An event of type “unload” is present in the document’s relevant global object’s [event listener list](https://dom.spec.whatwg.org/#eventtarget-event-listener-list) and if the document’s _unload salvageable opt-in state_ is false
5. If the user agent decided not to keep the document according to the previous step, set the document’s salvageable state to false.

## IANA considerations patch

Add an section on [IANA considerations](https://html.spec.whatwg.org/multipage/iana.html#iana)

## 17.X `BFCache-Opt-In`

This section describes a header for registration in the Permanent Message Header Field Registry. [[RFC3864]](https://html.spec.whatwg.org/multipage/references.html#refsRFC3864)

**Header field name:**

BFCache-Opt-In

**Applicable protocol:**

http

**Status:**

**standard**

Author/Change controller:

WHATWG

**Specification document(s):**

This document is the relevant specification.

**Related information:**

None.
