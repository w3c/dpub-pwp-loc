
# Locator structure issues

## The problem

Let us clarify what the problem we try to solve... (The URI-s below are slightly different than the ones in the [writeup](https://github.com/w3c/dpub-pwp-loc/blob/gh-pages/locators.md) to emphasize the differences.)

Let us suppose we have:

* A PWP (denoted by **P**) that includes the resource for an image of the Mona Lisa.
* **P** has a set of metadata associated to it. That set will be referred to by **M**. (The format and syntax is to be defined later.)
   * It must be emphasized that there is no requirement of having a one to one correspondence between **M** as a conceptual entity and a specific file, like a manifest file. A manifest file _may_ contain other type of information that one would not consider as metadata; also, elements of **M** may be combined from different sources (e.g., different manifest files, information conveyed through HTTP Responses, etc.)
* **P** is published on the server **S**, and it is published in *at least one* of the different states (“at least one” insofar as there is no requirement that the publisher publishes all these states, it may choose to publish only one):
   * In an “unpacked” state, i.e., as a hierarchy of files on the server. The “top level” of this unpacked state is available through the URL `http://book.org/books/1/`: this is the (absolute) Locator of **P** *in this state*. Let us refer to this as **Lu**.
     * in this setup, the image of the Mona Lisa has the URL `http://book.org/books/1/img/mona_lisa.jpg`. This is the absolute locator of the image *in this state*.  
   * In a “packed” state, namely as part of a, say, zip file (but with a PWP-specific media type). This is available through the URL `http://book.org/packed-books/1/package.pwp`: this is the (absolute) Locator of **P** *in this state*. Let us refer to this as **Lp**.
     * (For the sake of simplicity we can assume that the zip file reflects, internally, the same file structure as the unpacked state. If not, some of the URL translations in the **M** may become more complicated; that is to be specified, again, at some later point in time.)
   * There may be other states, mostly similarly packed states but using a different archiving technology (e.g., tar.gz). Their treatments are similar; we will not consider them in what follows because it does not change the various considerations.
* The PWP is assigned a separate, “canonical”  URL `http://book.org/published-books/1`. This is an *(absolute) Locator*, as opposed to an abstract identifier, i.e., it can be dereferenced via an HTTP request and should return some information. Note that, as the example shows, the exact URL string may be structurally *different* from both **Lu** and **Lp** (i.e., it is not necessarily a substring of one or the other). This URL is the *Canonical Locator of* **P**; let us refer to this as **L**.
	* reflecting the unpacked state in terms of (file) structure the *Canonical Locator* of the Mona Lisa image is `http://book.org/published-books/1/imgs/mona_lisa.jpg`
    * note that **L** _may_ be identical to **Lu**, but not necessarily

The goal of the PWP is that (quoting from the PWP draft):

“In this vision, the current format- and workflow-level separation between offline/portable and online (Web) document publishing is diminished to zero. These are merely two dynamic manifestations of the same publication: content authored with online use as the primary mode can easily be saved by the user for offline reading in portable document form.”

The translation, in practical terms, is that, whenever possible, the absolute locator **L**  should be used when referring to **P** in, e.g., annotations. This is also true URL-s derived from **L** like `http://book.org/published-books/1/img/mona_lisa.jpg`.

## Reminders

Some things we may already have an agreement on:

* **M** contains a reference to **L**, and **M** is also available in all the different states of **P**.
* It is to be expected that a client will need to include/use a PWP Processor of some sort.
  * E.g., it can be a specific reading app, containing both the PWP Processor and the Reading system; or it can be an extension/script of some sort in a browser whereby the script implements the PWP Processor and the Reading System tries to push all rendering on the browser's core engine, etc.
* We should aim for a model whereby the Reading System operates exclusively in terms of Canonical Locators and all mapping to and from that conceptual view of Locators to specific, non-canonical locators should be done by the PWP Processor.

> *Note*: Although our discussions should be technology independent, it is clear that this aspect of a PWP Processor, more exactly the implementation strategy thereof, resembles that of Service Workers. In other words, the more detailed description of a PWP Processor will have to define such characteristics on an abstract level.)

## The questions (and attempted answers)

Two fundamental (and related) questions we have to answer:

* **Q1**: What is the answer of **S** to a `HTTP Get http://book.org/published-books/1` request?
* **Q2**: How does a PWP Processor accesses the Mona Lisa image when `http://book.org/published-books/1/img/mona_lisa.jpg` is requested by the Reading System? (This, essentially, the **Q4** in the writeup.)

### Possible answer to **Q2**

Let us make the following assumptions:

1. The PWP Processor has access to the information in **M**.
2. As a consequence, **M** contains the list of states (and their Locators) that are available on **S**. In other words, the PWP Processor “knows” **Lp** and **Lu**, together with their media types.
3. In some way or other, the PWP Processor also has a *preference*, i.e., a priority among the different states.
   * whether this preference is based on some local consideration, like network speed, PWP size, etc., or is set by the user, is a different problem that will require separate considerations.)
   * obviously, if the PWP Processor already has the cached publication, than that will probably prevail (modulo cache state) and there may be no HTTP request in the first place. The question really refers to the situation of a first access.

If these assumptions are correct, then the answer to **Q2** becomes easy:

1. Based on the information on **L** (which is available in **M**) the processor can derive a Relative Locator for the image, i.e., `img/mona_lisa.jpg`
2. Based on the preference and the values of **Lp** and **Lu** the Processor can access the image:
   * If the preferred state is the packed one, then **Lp** accessed, unpacked, and the image is localized within the unpacked content (and that usually means using the relative URL as some sort of a file system path within that unpacked content)
   * If the preferred state is the unpacked one, then the locator of the image is constructed by using the value of **Lu** and the Relative Locator, yielding `http://book.org/books/1/img/mona_lisa.jpg`.

> *Note*: There may be “smarter” PWP Processors that make use local facilities like caching, but those do not modify these conceptual approaches.

### Possible answer to **Q1**

For the previous mechanism to work, the following statement, partially answering **Q1** seems to be the essential feature:

* The answer to `HTTP Get http://book.org/published-books/1` *must* make **M** available to the PWP Processor  

However, there is no one, and only one, way to achieve that. Instead, a PWP Processor must be prepared to some alternatives. The final list of those alternatives should be part of a more precise specification for a PWP; some of the possible variations are as follows. The response of **S** (i.e., the answer to **Q1**) is:

* **M** itself (serialized as a JSON file, and RDFa+HTML file, etc.); or
   * it may be important to return, as part of the HTTP response, a distinctive media type for the serialization of **M**
* a package in some predefined PWP format that _must_ include a manifest containing **M** (or a subset thereof); or
* an HTML, SVG, or other resource, representing, e.g., the cover page of the publication, with **M** (or a subset thereof) provided through the [`Link`](https://tools.ietf.org/html/rfc5988) header of the HTTP Response; or
* an (X)HTML file containing the `<link>` element referring to **M** (or a subset thereof)

These possibilities are not mutually exclusive. Each returned information, by itself, may be incomplete (hence the remark 'subset thereof') and a combination of those, or other possibilities, may provide the complete **M**.

> A typical example may be when the manifest file returned from the packed state would not include **Lu**; this may be returned via the HTTP Response LINK header

> **Note:** The relative priorities of these alternatives must be specified.

Whilst the statements above would be defined normatively in a specification of PWP, how exactly to set up **S** to achieve these should be left open. Depending on the possibilities and priorities of a server, there are indeed several conceivable setups. We list some of those below for informative purposes. For example, when `http://book.org/published-books/1` is requested,

* **S** may return the complete **M**. Ie, the publisher publishes the metadata for all its publications at one place, referring to the various, available, states on its site.
* **S** returns the index file in HTML of the unpacked state (i.e., using **Lu**) with the relevant `<link>` element
  * this can be achieved, for example, by setting a symbolic link (in UNIX terms) on **S** pointing from `/published-books/1` to `/books/1`
* If the various states are published in the same directory, i.e., for example, if **Lu** is set to `http://ex.org/books/xxx/`, **Lp** is set to `http://ex.org/books/xxx.pwp` and **L** is announced as `http://ex.org/books/xxx` (note the missing trailing `/`!) then **S** may have its own logic deciding which state to return with the corresponding **M**. The logic may be based on:
  * the local preferences of the publisher hardwired in the behavior of **S**
  * if the PWP Processor accompanies its request with the [`Accept`](https://tools.ietf.org/html/rfc7231#section-5.3.2) header, indicating the states that are acceptable by the server and with what priority, **S** may be set up to honor that part of the request (i.e., implementing content negotiations)

> *Note*: there may be other scenarios on the server side; it is out of scope for PWP to define those.
