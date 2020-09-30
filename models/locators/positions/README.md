# Positions List

While reading a publication, users may need to reference or access a specific position.

While some formats have an implicit (image based publications) or an optional explicit option (EPUB 3.x) to express such references, clients need references even for publication formats that do not provide anything.

The goal of this service is to provide such references, no matter what the original publication format contains.


## Manifest

The list is a service exposed in the manifest using a Link Object. 

It can be identified by: 

- its media-type: `application/vnd.readium.position-list+json`
- its relationship: `http://readium.org/position-list`


```json
{
  "href": "https://example.com/list",
  "type": "application/vnd.readium.position-list+json",
  "rel": "http://readium.org/position-list"
}
```

## Usage

A streamer will usually be responsible for handling that service and can decide whether the list is computed in realtime or in advance.

A client may request the full list of positions for the publication by sending a simple GET request:

```json
{
  "total": 2,
  "positions": [
    {
      "href": "http://example.com/resource",
      "type": "application/xhtml+xml",
      "locations": {
        "fragment": "partialcfi(/4[body01])",
        "position": 1,
        "progression": 0.0
      }
    },
    {
      "href": "http://example.com/resource",
      "type": "application/xhtml+xml",
      "locations": {
        "fragment": "partialcfi(/18[body02])",
        "position": 2,
        "progression": 0.57289
      }
    }
  ]
}
```

## Syntax

| Key  | Definition | Format |
| ---- | ---------- | ------ | 
| total  | Contains the total number of positions in a given publication.  | Integer |
| positions  | Contains one or more Locator Objects.  | Locator Object |

The Locator Object contains one or more locations, [as defined in the Locators document](../README.md#the-locator-object).

## Navigator

In order to facilitate the support of position references in full apps, navigators should provide an easy way to obtain:

* the total number of positions in a publication
* the current position currently being displayed
* the number of positions for a specific resource along with the start position for that resource


