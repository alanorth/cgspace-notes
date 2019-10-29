+++
title = "CGSpace CG Core v2 Migration"
date = 2019-10-28T13:27:35+02:00
description = "Possible changes to CGSpace metadata fields to align more with DC, QDC, and DCTERMS as well as CG Core v2."
categories = ["Notes"]
tags = ["Migration"]
url = "cgspace-cgcorev2-migration"

+++

Possible changes to CGSpace metadata fields to align more with DC, QDC, and DCTERMS as well as CG Core v2.

With reference to [CG Core v2 draft standard](https://agriculturalsemantics.github.io/cg-core/cgcore.html) by Marie-Angélique as well as [DCMI DCTERMS](http://www.dublincore.org/specifications/dublin-core/dcmi-terms/).

<!--more-->

- [Proposed Changes](#proposed-changes)
- [Fields to Create](#fields-to-create)
- [Fields to Delete](#fields-to-delete)
- [Implementation Progress](#implementation-progress)

## Proposed Changes
As of 2019-10-29 the scope of the changes includes the following fields:

- cg.creator.id→cg.creator.identifier
  - ORCID identifiers
- dc.format.extent→dcterms.extent
- dc.date.issued→dcterms.issued
- dc.description.abstract→dcterms.abstract
- dc.description→dcterms.description
- dc.description.sponsorship→cg.contributor.donor
  - values from CrossRef or Grid.ac if possible
- dc.description.version→cg.peer-reviewed
- cg.fulltextstatus→cg.howpublished
  - CGSpace uses values like "Formally Published" or "Grey Literature"
- dc.identifier.citation→dcterms.bibliographicCitation
- cg.identifier.status→dcterms.accessRights
  - current values are "Open Access" and "Limited Access"
  - future values are possibly "Open" and "Restricted"?
- dc.language.iso→dcterms.language
  - current values are ISO 639-1 (aka Alpha 2)
  - future values are possibly ISO 639-3 (aka Alpha 3)?
- cg.link.reference→dcterms.relation
- dc.publisher→dcterms.publisher
- dc.relation.ispartofseries→dcterms.isPartOf
- dc.rights→dcterms.license
  - Using [SPDX license identifiers](https://spdx.org/licenses/) if possible
- dc.source→cg.journal
- dc.subject→dcterms.subject
- dc.type→dcterms.type
- dc.identifier.isbn→cg.isbn
- dc.identifier.issn→cg.issn
- cg.identifier.dataurl→cg.hasMetadata

The following fields are currently out of the scope of this migration because they are used internally by DSpace 5.x/6.x and would be difficult to change without significant modifications to the core of the code:

- dc.title (`IncludePageMeta.java` only considers DC when building pageMeta, which we rely on in XMLUI because of XSLT from DRI)
- dc.title.alternative
- dc.date.available
- dc.date.accessioned
- dc.identifier.uri (hard coded for Handle assignment upon item submission)
- dc.description.provenance
- dc.contributor.author (`IncludePageMeta.java` only considers DC when building pageMeta, which we rely on in XMLUI because of XSLT from DRI)

## Fields to Create
Make sure the following fields exist:

- [x] cg.creator.identifier (242)
- [x] cg.contributor.donor (243)
- [x] cg.peer-reviewed (244)
- [x] cg.howpublished (245)
- [x] cg.journal (246)
- [x] cg.isbn (247)
- [x] cg.issn (248)
- [x] cg.hasMetadata (249)

## Fields to delete
Fields to delete after migration:

- [ ] cg.creator.id
- [ ] cg.fulltextstatus
- [ ] cg.identifier.status
- [ ] cg.link.reference
- [ ] cg.identifier.dataurl

## Implementation Progress
Tally of the status of the implementation of the new fields in the CGSpace `5_x-cgcorev2` branch.

| Field Name | migrate-fields.sh | Input Forms | XMLUI Themes¹ | dspace.cfg | Discovery | Atmire Modules | Crosswalks |
| ---------- | :---------------: | :---------: | :-----------: | :--------: | :-------: | :------------: | :--------: |
cg.creator.identifier | ✓ | ✓ | ✓ | - | ✓ | ✓ | |
dcterms.extent | ✓ | ✓ | - | - | - | - | |
dcterms.issued | ✓ | ✓ | ? | ✓ | ✓ | ✓ | |
dcterms.abstract | ✓ | ✓ | ✓ | ✓ | ✓ | - | |
dcterms.description | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | |
cg.contributor.donor | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | |
cg.peer-reviewed | ✓ | ✓ | ✓ | - | ✓ | - | |
cg.howpublished | ✓ | ✓ | - | - | - | - | |
dcterms.bibliographicCitation | ✓ | ✓ | ✓ | - | - | ✓ | |
dcterms.accessRights | ✓ | ✓ | ✓ | - | ✓ | ✓ | |
dcterms.language | ✓ | ✓ | ✓ | - | ✓ | ✓ | |
dcterms.relation | ✓ | ✓ | ✓ | - | - | - | |
dcterms.publisher | ✓ | ✓ | - | - | ✓ | ✓ | |
dcterms.isPartOf | ✓ | ✓ | - | ✓ | ✓ | ✓ | |
dcterms.license | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | |
cg.journal | ✓ | ✓ | - | - | ✓ | ✓ | |
dcterms.subject | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | |
dcterms.type | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | |
cg.isbn | ✓ | ✓ | - | - | - | ✓ | |
cg.issn | ✓ | ✓ | - | - | - | ✓ | |
cg.hasMetadata | ✓ | ✓ | ✓ | - | ✓ | ✓ | |

There are a few things that I need to check once I get a deployment of this code up and running:

- Assess the XSL changes to see if things like `not(@qualifier)]` still make sense after we move fields from DC to DCTERMS, as some fields will no longer have qualifiers
- Do I need to edit crosswalks that we are not using, like [MODS](https://wiki.duraspace.org/display/DSDOC5x/DSpace+AIP+Format#DSpaceAIPFormat-MODSSchema)?
- There is potentially a lot of work in the OAI metadata formats like DIM, METS, and QDC (see `dspace/config/crosswalks/oai/*.xsl`)

------
¹ Not committed yet because I don't want to have to make minor adjustments in multiple commits. Re-apply the gauntlet of fixes with the sed script:

```
$ find dspace/modules/xmlui-mirage2/src/main/webapp/themes -iname "*.xsl" -exec ./cgcore-xsl-replacements.sed {} \;
```
