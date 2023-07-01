+++
title = "Harmonization of CG Core Output Types"
date = 2021-02-21T13:27:35+02:00
description = "Proposed changes to CG Core types after review of several CGIAR repositories."
categories = ["Notes"]
tags = ["Migration"]
url = "cgcore-types-harmonization"
draft = true

+++

Proposed changes to the CG Core controlled vocabulary for output types after review of actual usage by several CGIAR open access repositories.

With reference to [CG Core v2 draft standard](https://agriculturalsemantics.github.io/cg-core/cgcore.html) by Marie-Angélique as well as [DCMI DCTERMS](http://www.dublincore.org/specifications/dublin-core/dcmi-terms/).

<!--more-->

- [Proposed Changes](#proposed-changes)
  - [Out of Scope](#out-of-scope)
- [Implementation Progress](#implementation-progress)

## Proposed Changes
As of 2021-01-18 the scope of the changes includes the following fields:

- cg.creator.id→cg.creator.identifier
  - ORCID identifiers
- dc.format.extent→dcterms.extent
- dc.date.issued→dcterms.issued
- dc.description.abstract→dcterms.abstract
- dc.description→dcterms.description
- dc.description.sponsorship→cg.contributor.donor
  - values from CrossRef or Grid.ac if possible
- dc.description.version→cg.reviewStatus
- cg.fulltextstatus→cg.howPublished
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
- dc.relation.ispartofseries will be split into:
  - series name: dcterms.isPartOf
  - series number: cg.number
- dc.rights→dcterms.license
  - Using [SPDX license identifiers](https://spdx.org/licenses/) if possible
- dc.source→cg.journal
- dc.subject→dcterms.subject
- dc.type→dcterms.type
- dc.identifier.isbn→cg.isbn
- dc.identifier.issn→cg.issn
- cg.targetaudience→dcterms.audience

### Out of Scope
The following fields are currently out of the scope of this migration because they are used internally by DSpace 5.x/6.x and would be difficult to change without significant modifications to the core of the code:

- dc.title (`IncludePageMeta.java` only considers DC when building pageMeta, which we rely on in XMLUI because of XSLT from DRI)
- dc.title.alternative
- dc.date.available
- dc.date.accessioned
- dc.identifier.uri (hard coded for Handle assignment upon item submission)
- dc.description.provenance
- dc.contributor.author (`IncludePageMeta.java` only considers DC when building pageMeta, which we rely on in XMLUI because of XSLT from DRI)
