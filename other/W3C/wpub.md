# W3C Web Publication Manifest

## Overview

The purpose of this document is to take a look at a snapshot of the W3C Web Publication Manifest (which we'll reference as "W3C Manifest") draft (December 2017, First Public Working Draft) and compare it to the Readium Web Publication Manifest ("Readium Manifest").

There are a number of differences in how these documents are designed that are worth pointing out:

- the Readium Manifest is based on early work started (but never completed) by the EPUB 3.1 WG
- it's meant to be a "lossless" format for EPUB 2.x and 3.x, which means that all metadata and properties from various EPUB versions can be expressed in the Readium Manifest as well
- in addition to EPUB, the Readium Manifest is also meant to work well with comics and audiobooks
- one of the design goal of the Readium Manifest is to create a syntax that works well for hypermedia in general, not just publications
- at the core of every element defined in the Readium Manifest, there's built-in extensibity

The W3C Manifest has less ambitious goals:

- it's mostly a collection model for the Web with a few additional metadata
- it doesn't attempt to cover all the EPUB metadata/properties (for instance FXL specific info such as orientation, spreads or left/right page, which are also useful for comics)
- it doesn't define a generic hypermedia format with built-in extensibility

## 1. Metadata

In the current draft, the W3C Manifest doesn't provide any serialization for metadata. Instead it defines a minimal infoset that explicitly states that most metadata should be external to the manifest.

The Readium Manifest has a very different approach, since it's based on JSON-LD and schema.org and provides extensive metadata in addition to built-in extensibility using:

- additional schema.org elements
- or the definition of new JSON-LD context documents

The metadata listed in the Readium Manifest are therefore meant to be compatible with any JSON-LD enabled User Agent (including search engine crawlers) and a RDF graph can be extracted using the context documents.

### 1.1. Title

The title is the only required element in the Readium Manifest, but that's not the case for the W3C Manifest where the User Agent can fallback to resources or default options to generate a document.

The nature of the title is not described in [the infoset from the W3C draft](https://w3c.github.io/wpub/#wp-title).

[In the Readium Manifest](https://github.com/readium/webpub-manifest/tree/master/contexts/default#title), the title is mapped to schema.org and it can be:

- a single string
- an array of localized strings, using BCP 47 language tags

*Examples*

```
"title": "Moby-Dick"
```

```
"title": {
  "fr": "Vingt mille lieues sous les mers",
  "en": "Twenty Thousand Leagues Under the Sea",
  "ja": "海底二万里"
}
```

In addition to a title, the Readium Manifest also defines a different sortable string for User Agents, along with a subtitle element. These elements are mapped to schema.org as well.

*Examples*

```
"title": "Flatland",
"subtitle": "A Romance of Many Dimensions"
```

```
"title": "A Tale of Two Cities",
"sort_as": "Tale of Two Cities, A"
```

The W3C infoset does not mention subtitles and the ability to sort titles.

### 1.2. Address

The W3C infoset requires the W3C Manifest to contain a reference to the WP Publication Address:
> A Web Publication's address is a URL that represents the primary entry point for the Web Publication. This URL MUST resolve to an HTML document.

The Readium Manifest has no such requirement, but it contains a generic link element that could be used for that purpose.

The IANA link registry contains the following definition for `start` which seems like a good fit for this use case:
> Refers to the first resource in a collection of resources.


The Readium Manifest has a single requirement for links: it must contain a `self` link that provides the canonical URI for the manifest.

If we add a `start` link as well, the `links` element would look like this:

```
"links": [
  {
    "href": "manifest.json",
    "type": "application/webpub+json",
    "rel": "self"
  },
  {
    "href": "/publication",
    "type": "text/html",
    "rel": "start"
  }
]
```

### 1.3. Canonical Identifier

[The W3C infoset](https://w3c.github.io/wpub/#wp-canonical-identifier) has an optional identifier in addition to the WP Address, called the canonical identifier:
> A Web Publication's canonical identifier is a unique identifier that resolves to the preferred version of the Web Publication.

The Readium Manifest contains a dedicated `identifier` element [in its default context](https://github.com/readium/webpub-manifest/blob/master/contexts/default/README.md#identifier) that serves the exact same purpose:

```
"identifier": "urn:isbn:9780142437247"
```

In both specifications, this canonical identifier is optional (SHOULD statement for Readium, no specific statement for the W3C infoset).

In the Readium Manifest, the canonical identifier MUST be a URI, while this requirement is a little bit different for the W3C infoset:
> The canonical identifier SHOULD be an address, but, if not, it MUST be possible to make a one-to-one mapping to an address (e.g., a DOI can be resolved to a URL via a DOI resolver).

In the Readium Manifest, `identifier` is mapped to `@id` in JSON-LD, which turns this canonical identifier into the subject of many RDF triples.

### 1.4. Creators

[The W3C infoset](https://w3c.github.io/wpub/#wp-creators) does not explicitly indicates if the presence of creators is a requirement:
> Creators are the individuals or entities responsible for the creation of the Web Publication.

It recommends listing the role as well, which might be a hint that a creator and its role are two distinct information in its infoset:
> The role the creator played in the creation of the Web Publication SHOULD also be specified (e.g., 'author', 'illustrator', 'translator').

[The Readium Manifest](https://github.com/readium/webpub-manifest/tree/master/contexts/default#contributors) has a fairly long list of contributors (and not creators), all based on a mapping to schema.org: `author`, `translator`, `editor`, `artist`, `illustrator`, `letterer`, `penciler`, `colorist`, `inker` and `narrator`.

It also has a generic `contributor` element that behaves like EPUB 2.x, with a `role` based on [MARC relator codes](https://www.loc.gov/marc/relators/relaterm.html) in addition to strings.

These contributors elements are fairly consistent with how `title` works and can be expressed using:

- a single string 

```
"author": "James Joyce"
``` 
- an array of strings

```
"artist": ["Shawn McManus", "Colleen Doran", "Bryan Talbot"]
``` 
- an object or array of objects... 

*... with a sortable string*

```
"author": {
  "name": "Marcel Proust",
  "sort_as": "Proust, Marcel"
}
```

*... and/or localized strings*

```
"author": {
  "name": {
    "ru": "Михаил Афанасьевич Булгаков",
    "en": "Mikhail Bulgakov",
    "fr": "Mikhaïl Boulgakov"
  }
}
``` 

*... and/or an identifier*

```
"author": {
  "name": "Jules Amédée Barbey d'Aurevilly",
  "sort_as": "Barbey d'Aurevilly, Jules Amédée",
  "identifier": "http://isni.org/isni/0000000121317806"
}
``` 

This approach is consistent with EPUB 2.x and 3.1 but it makes a larger number of roles first class citizens, provides the ability to localize a contributor's name in more languages and adds the ability to uniquely identify contributors.

### 1.5. Language

[The W3C infoset](https://w3c.github.io/wpub/#wp-language) specifies the following requirement for a language:
> When specified, the language MUST be a tag that conforms to BCP 47.

This is very similar to the default context for the Readium Manifest:

- both specifications require the use of BCP 47 language tags
- the Readium Manifest explicitly allows one or more tags, while the W3C infoset remains a little unclear about that
- the Readium Manifest maps `language` to `http://schema.org/inLanguage`

```
"language": "en"
```

```
"language": ["en", "fr", "ja"]
```

In both specifications, the presence of a language is optional.

### 1.6. Publication and Last Modification Dates

The W3C infoset contains references to a [publication date](https://w3c.github.io/wpub/#wp-pub-date) and a [last modification date](https://w3c.github.io/wpub/#wp-mod-date).

The Readium manifest also has the same two properties in its default context: `modified` and `published`, mapped respectively to `http://schema.org/dateModified` and `http://schema.org/datePublished`. 

```
"modified": "2016-02-22T11:31:38Z"
```

```
"published": "2016-09-02"
```

They're both required to use an ISO 8601 time and date.

### 1.7. Base Direction

Both manifests contain a dedicated element for the reading direction of the publication.

In the Readium Manifest, this is not currently mapped to a schema.org element since no corresponding element could be identified.

### 1.8. Privacy Policy

Unlike [the W3C infoset](https://w3c.github.io/wpub/#wp-privacy), the Readium Manifest does not contain any reference to a privacy policy.

The IANA link registry contains the `privacy-policy` rel value with the following definition:
> Refers to a privacy policy associated with the link's context.

The Readium Manifest can simply rely on its `links` element and this rel value to include a privacy policy as well:

```
"links": [
  {
    "href": "/terms",
    "type": "text/html",
    "rel": "privacy-policy"
  }
]
```

### 1.9. Accessibility

[The W3C infoset](https://w3c.github.io/wpub/#wp-a11y) is a bit vague regarding accesibility requirements, but it explicitly mentions schema.org properties:
> Editor's Note:
> 
> How to express accessibility metadata remains to be determined. This could be a grouping of properties from schema.org, for example, or could be split out into a list of individual properties.

The Readium Manifest does not explicitly list accesibility properties [in its default context document](https://github.com/readium/webpub-manifest/tree/master/contexts/default), but it contains a reference to schema.org which means that all of the accessibility properties from schema.org can be referenced without any additional extension.

## 2. Collections

The default reading order and the resource list are two core concept shared by the W3C and Readium manifests.

### 2.1. Generic Collection Model

Unlike the W3C infoset, the Readium Manifest defines a generic collection model, used by the default reading order as well as the resource list.

All collections are identified by their role (such as `spine` and `resources`) and the Readium Manifest can be extended by defining new collection roles.

Collections usually contain an array of links (see section 3 for additional info about `links` and the Link Object), and can optionally contain sub-collections and metadata as well.

### 2.2. Default Reading Order

The Readium Manifest calls its default reading order the `spine`, inspired by the term used in EPUB 2.x/3.x.

One major difference between the W3C infoset and the Readium specification is expressed in the following paragraph:
> The default reading order is either specified directly in the manifest or in an HTML `nav` element. In the latter case, the manifest MUST provide a link to the Web Publication resource that contains the nav element, with a fragment identifier that specifically identifies it.

Unlike the W3C infoset, the Readium Manifest requires the default reading order to be explicitly expressed in the manifest, not indirectly through an HTML document.

In the context of Readium-2, the Readium Manifest is meant to make the life of reading system developers easier and avoid situations where things are unclear. 

Having an explicit default reading order in the manifest is consistent with these goals: there's no need to fetch an additional HTML document, parse HTML in addition to JSON or deduplicate references to resources in a `nav` element.

In my opinion, "guessing" a reading order through an HTML document and a `nav` element is also inconsistent with the design goals of the W3C Manifest as well:
> A Web Publication is a collection of one or more resources, organized together through a manifest into a single logical work with a default reading order. The Web Publication is uniquely identifiable and presentable using Open Web Platform technologies.

The concept of a collection of Web resources is the main (and maybe only) innovation of the Web Publication WG. If the manifest can't always express this information on its own, it undermines the main reason which justifies its existence in the first place.

### 2.3. Resource List

Both specifications highly recommend including a list of resources used in the processing and rendering of the Web Publication.

In the Readium Manifest, the resource list is identified by the `resource` collection role and MUST NOT reference resources already listed in the default reading order (`spine`).

### 2.4. TOC and Additional Navigation

The W3C infoset mentions the table of contents multiple times, but does not explicitly states how it is identified or expressed.

The Readium Manifest provides multiple options for expressing this information:

- a resource in the resource list can be identified as a TOC using the `contents` rel value
- a minimal TOC can be extracted from the `title` of linked resources in the default reading order
- [the EPUB extension for the Readium Manifest](https://github.com/readium/webpub-manifest/blob/master/extensions/epub.md#collection-roles) also defines a number of additional collection roles: `landmarks`,  `loa`, `loi`, `lot`, `lov`, `page-list` and `toc`

Once again, this is more consistent with the Readium Manifest stated goal of being a "lossless" alternative serialization for EPUB 2.x/3.x.

## 3. Links

The W3C infoset never directly mentions links as a requirement, but there are multiple indirect references anyway, including but not limited to:

- providing the WP Address
- linking to the privacy policy
- linking to the HTML document containing the default reading order
- linking to the TOC

While the serialization has yet to be defined for the W3C infoset, a generic link element could be more efficient than creating dedicated elements for all the use cases listed above.

The Readium Manifest approach to links is to define two building blocks:

- the `links` element, used as a generic link element at a publication level
- the Link Object, used in the `links` element as well as the default reading order (`spine`) and the resource list (`resource`)

The Readium Manifest has a single requirement for `links`: the presence of a `self` link, containing the canonical location for the manifest.

```
{
  "rel": "self",
  "href": "http://example.org/manifest.json",
  "type": "application/webpub+json"
}
```

Link Objects are fairly powerful and provide a number of hypermedia controls such as:

- a required media type
- one or more `rel` values, based on the IANA link registry
- the ability to use URI Templates as well as URIs (using the `templated` flag to identify them)

Using these hypermedia controls, links can be useful for many use cases, for instance:

- the discovery of additional services (search, annotations, dictionaries)
- alternative representations in another publication format

```
"links": [
  {
    "rel": "self", 
    "href": "http://example.org/manifest.json", 
    "type": "application/webpub+json"
  },
  {
    "rel": "alternate", 
    "href": "http://example.org/publication.epub", 
    "type": "application/epub+zip"
  },
  {
    "rel": "search", 
    "href": "http://example.org/search?q={searchTerms}", 
    "type": "text/html", 
    "templated": true
  }
]
```

To provide full compatibility with EPUB 2.x/3.x, the Link Object can also contain [a number of properties](https://github.com/readium/webpub-manifest/blob/master/properties.md) such as (but not limited to):

- `page` to indicate how a resource should be displayed in a synthetic spread
- `orientation` to indicate the preferred orientation for displaying a resource

## Conclusion

While there are a few important differences (the requirement for a title and an explicit default reading order), the Readium Manifest is essentially a valid W3C Manifest as well.

Everything listed in the W3C infoset is either supported natively by the Readium Manifest or easily added to it using links and the IANA link registry.

Given the fact that the Readium Manifest has better forward compatibility with the EPUB specification, JSON-LD based metadata mapped to schema.org, built-in extensibility and hypermedia controls, it is a superset of the current infoset for the W3C Manifest as well.

In 2018 and beyond, the WP WG will have to work on serializing their infoset in JSON. Based on this comparaison, the Readium Manifest should be seriously considered by the WP WG as a candidate as well.


