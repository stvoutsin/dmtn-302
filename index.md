# IVOA Identifier Usage at the Rubin Observatory

```{abstract}
The IVOA define a standard URI scheme to reference individual datasets and registry resources.
In this document we will define how these IVOA Identifiers will be formed for Butler datasets and object catalog IDs, and describe the service and collection registry records published by Rubin Observatory and the IVOIDs assigned to them.
```

DOI: [10.71929/rubin/2583848](https://doi.org/10.71929/rubin/2583848)

## Introduction

The IVOA Identifier (IVOID) standard {cite:p}`2016ivoa.spec.0523D` provides a mechanism for publishers to provide unique identifiers to individual datasets using an `ivo` URI scheme.
The IVOID consists of a registry part that must be registered in VO Registry, and a local part that defines the specific resource.
The registry part consists of a naming authority (effectively an internet location corresponding to the `netloc` field) and a resource key (corresponding to the `path`).
The local part corresponds to the `query` component of a URI and can additional be fine-tuned by the use of a fragment.
This document will address the naming authority, resource key, and query parts of the URI.

IVOIDs are used in the ObsCore standard, in particular, for the following fields: `obs_publisher_did`, `obs_creator_did`, and `publisher_id`.  The roles of these fields, and how we will use them, are discussed below.
While IVOIDs do provide a standard means of labeling datasets, with the intention of stability across the lifetime of
the publisher, they are not equivalent to, and are not substitutes for, the use of DOIs for LSST entities and datasets.
Those are discussed elsewhere, primarily in {cite:p}`DMTN-318`.

## Naming Authority

There is no requirement that the naming authority component correspond to a public DNS name, and therefore not to a web site either, since the value is looked up in the VO Registry.
Indeed, IVOA recommendation is to deliberately avoid using a standard DNS name,
with examples given such as `ivo://org.gavo.dc/` (reversing the internet address) and `ivo://adil.ncsa` (publishing organization and archive).

This suggested that Rubin should consider names such as:

* `lsst.rubin`
* `rubin.lsst`
* `org.lsst`
* `org.rubinobservatory`
* `org.rubinobs`

Given that there is potential for a non-LSST survey in the future and given the naming convention we use for Jira hosting we have decided that Rubin will use `org.rubinobs` as the naming authority.
The naming authority will represent data published by the Observatory and/or through its data access services.

## Resource Key

The resource key (in effect, the `path`) is also part of the lookup key used to access the IVOA Registry,
and thus allows us to segregate our registry entries, if we wish.
It would be possible to do what GAVO does and use a single uniform value of `~` to indicate that all our resources are available under a single registry entry.
We could also segregate based on type of dataset or data release.

Because of the likely growth in the number and variety of Registry resources from data release to data release,
closely related to the growth in the number of service endpoints,
we feel that enabling registry lookups per data release is essential for the documentation of our data services.

### "Creator" and "Publisher" Resource Keys

The most important near-term use of IVOID in our data services is in the context of ObsCore-based image metadata services (SIAv2 {cite:p}`2015ivoa.spec.1223D` and ObsTAP).
In the ObsCore standard {cite:p}`2017ivoa.spec.0509L`, there are three uses of IVOID-valued attributes in the data model:

* `publisher_id`: "The IVOA ID for the data provider as defined in the Spectrum DM"
* `obs_publisher_did`: "IVOA dataset identifier (Plante and al. 2007) for [a] published data product" (site-dependent)
* `obs_creator_did`: "IVOA dataset identifier given by [a dataset's] creator"

For `publisher_id` we will normally use simply `ivo://org.rubinobs` without an additional resource key.
This represents publication by the Observatory through data services that it controls or that have chosen, by agreement with the Observatory, to represent themselves as affiliated publishers (e.g., IDACs).

For the latter two attributes, which apply to individual datasets, the following two paragraphs
from the ObsCore v1.1 standard are relevant and provide additional clarification.
From section 4.5:

> "The `obs_publisher_did` column contains the IVOA dataset identifier (Plante and al. 2007) for the published data product.
> **This value must be unique within the namespace controlled by the dataset publisher** (data center).
> The value will also be globally unique since each publisher has a unique IVOA registered publisher ID.
> The same dataset may however have more than one publisher dataset identifier if it is published in more than one location; the creator DID, if defined for the given dataset, would be the same regardless of where the data is published.
> The returned `obs_publisher_did` for a static data product should remain identical
through time for future reference."

and from section B.3:

> "We ... distinguish two datasets curated by two different services (archives) but originating from the same creator.
> When broadcasting a query to multiple servers, the response may contain multiple copies of the same dataset, with a unique `obs_creator_did` but possibly different `obs_publisher_did` (given by the data provider)."

With this in mind we allow for including both the DAC and the data release in the resource key.

Specifically, we plan to issue `obs_creator_did`-style IVOIDs of the form:

* `ivo://org.rubinobs/lsst-dp1/datasets`
* `ivo://org.rubinobs/lsst-pp/datasets`
* `ivo://org.rubinobs/lsst-dr1/datasets`

as well as `obs_publisher_did`-style IVOIDs of the form:

* `ivo://org.rubinobs/usdac/lsst-dp1/datasets`
* `ivo://org.rubinobs/usdac/lsst-pp/datasets`
* `ivo://org.rubinobs/usdac/lsst-dr1/datasets`

and for other DACs, e.g.,

* `ivo://org.rubinobs/ukdac/lsst-dp1/datasets`

Differentiating the project's, and partner, DACs in the resource key allows the Registry to represent choices
that DACs may make to serve different subsets of of the available data (resources) via different constellations of services.

DACs following this pattern *must* ensure that the `obs_publisher_did` and `obs_creator_did` for released data-product
datasets coming from the Observatory differ only by the presence or absence of the DAC field in the resource key,
so that the `obs_creator_did` can be programmatically obtained from the `obs_publisher_id` if necessary.

IDACs and others republishing LSST data may, alternatively, choose an entirely different pattern for
`obs_publisher_did` if it suits their site's needs, in which case they must use a naming authority
other than `org.rubinobs`.
(For example, the planned service of the public data from CADC might well be under CADC's naming authority
and be based on their existing CAOM-based mechanisms for assigning `obs_publisher_did` values.)

However, we will expect the original Observatory-assigned value for `obs_creator_did` to be preserved in all such cases.

We intend to ensure that LSST data will be queriable both by `obs_creator_did` and by `obs_publisher_did`,
though this may not be fully in place in the DP1 era.
We believe it will be substantially useful for users to be able to re-use `obs_creator_did` values across DACs.

## Service and Collection Registry Records

The IVOA Identifiers standard {cite:p}`2016ivoa.spec.0523D` requires that for any IVOID, the registry part (scheme, authority, and path before the `?`) must resolve to a record in an IVOA Registry.
For a per-object IVOID like `ivo://org.rubinobs/lsst-dp1/datasets?type=object&release=dp1&id=OBJECTID`, the registry part `ivo://org.rubinobs/lsst-dp1/datasets` must therefore be a published registry record.

Rubin Observatory will publish the following registry records.

**Infrastructure records:**

- `ivo://org.rubinobs`: IVOA naming authority record for Rubin
- `ivo://org.rubinobs/registry`: the Rubin IVOA publishing registry
- `ivo://org.rubinobs/org`: Rubin Observatory as an institution

**Service records:**

Service records describe physical service deployments and are named by deployment, since a single service may serve multiple datasets.

- `ivo://org.rubinobs/qserv-tap`: TAP service providing catalog access across all data releases stored in QServ
- `ivo://org.rubinobs/cutout`: SODA image cutout service

SODA services do not require per-dataset collection records in the registry.
SODA endpoints are generally discovered dynamically via DataLink responses from SIA and ObsCore queries, rather than through a registry lookup.

**Per-release records:**

For each data release, Rubin publishes a dataset identity record and one or more service-linked collection records, all sharing the same root release identifier:

- `ivo://org.rubinobs/lsst-dp1/datasets`: a `vs:DataResource` record serving as the resolvable target for the registry part of all per-object IVOIDs for DP1 (i.e. Butler datasets & HiPS surveys).
  This record is not designed to be referenced by any specific service and describes the data release as a collection entity.

- `ivo://org.rubinobs/lsst-dp1/catalogs`: a `vs:CatalogResource` record linking the DP1 catalog data to the TAP service.
  This record carries an auxiliary TAP capability pointing to the TAP service, an `IsServedBy` relationship to `ivo://org.rubinobs/qserv-tap`, and a `<tableset>` specific to DP1.
  It is the discoverable record for clients searching the registry for catalog data from this release.

- `ivo://org.rubinobs/lsst-dp1/sia`: a `vs:CatalogService` (SIAv2) record for image access for this release.
  Since SIA services are deployed per dataset with distinct URLs, the service record itself describes both the service and the collection and thus no separate `vs:CatalogResource` is needed.

The `/datasets` record will be the stable resolvable target for per-object IVOIDs (regardless of how the data is served), while catalog-specific metadata (i.e. tableset) and the TAP linkage is included in the separate `/catalogs` record that can be modified independently.

For DAC-specific publisher records, the same pattern applies under the DAC sub-path, e.g.:

- `ivo://org.rubinobs/ukdac/lsst-dp1/datasets`: `vs:DataResource` dataset collection record for DP1 at the UK DAC
- `ivo://org.rubinobs/ukdac/lsst-dp1/catalogs`: `vs:CatalogResource` for DP1 catalog data at the UK DAC
- `ivo://org.rubinobs/ukdac/lsst-dp1/sia`: SIAv2 service record for DP1 images at the UK DAC

## Query

The query part of the IVOID indicates how a specific dataset should be referenced.
For a Butler dataset (see e.g., {cite:t}`2022SPIE12189E..11J`) we would need the repository label and the dataset UUID.
This may result in duplication of data release information from the resource key but we do not see that as a problem.
The query string can be either a string that should be parsed into components or use the more explicit query string format.
Rather than using something like `?label/UUID` we have decided to use the more explicit form of `?repo=label&id=UUID`.

The intent is that a full IVOID for a dataset (e.g., from an ObsCore `obs_publisher_did`) should be usable to drive a Butler lookup of that dataset.
(This assumes backward compatibility, at least at the level of the ability to perform a `Butler.get()` operation, if not to run the pipelines.)
Note that UUIDs are already expected to remain stable across export/import of data from its "Butler of origin" to others, e.g., at remote DACs.

### Catalog Entries

There is no requirement for us to issue IVOA identifiers for individual catalog entries, but there is nothing to prevent this if we so desire.
Catalog IDs are meant to be unique for a given data release but are not necessarily unique across the entire lifetime of the survey for all ID types.

In a similar way to our handling of query strings for Butler datasets we propose a form of `ivo://org.rubinobs/lsst-dr1/catalogs?type=object&release=dr1&id=OBJECTID` for catalog entries.
Here type can be `object`, `source`, `diaobject`, `diasource`, or `forcedsource`.

### HiPS images

HiPS property files {cite:p}`2017ivoa.spec.0519F` are also required to declare an IVOID.
A single dataset type can be represented by multiple HiPS renderings, and therefore the scheme we are adopting is:

* `ivo://org.rubinobs/lsst-dp1/datasets?hips=<hips label>&type=<butler dataset type>`

## Combined Example

**Registry records**:

* `ivo://org.rubinobs/lsst-dp1/datasets`: `vs:DataResource` dataset collection record for DP1
* `ivo://org.rubinobs/lsst-dp1/catalogs`: `vs:CatalogResource` for DP1 catalog data, linked to the TAP service
* `ivo://org.rubinobs/lsst-dp1/sia`: `vs:CatalogService` (SIAv2) record for DP1 image access
* `ivo://org.rubinobs/qserv-tap`: TAP service record
* `ivo://org.rubinobs/cutout`: SODA image cutout service record

**Per-object IVOIDs** (`obs_creator_did` form):

* `ivo://org.rubinobs/lsst-dr1/datasets?repo=dr1&id=UUID`
* `ivo://org.rubinobs/lsst-dp1/catalogs?type=object&release=dp1&id=OBJECTID`
* `ivo://org.rubinobs/lsst-dp1/datasets?hips=color_gri&type=deep_coadd`

`obs_publisher_did`-style IVOIDs will be constructed by insertion of the DAC name in the resource key, e.g.:

* `ivo://org.rubinobs/usdac/lsst-dr1/datasets?repo=dr1&id=UUID`

The registry part of any per-object IVOID (i.e. the IVOID with the local part stripped) resolves to the appropriate collection record for that release.
Butler dataset and HiPS IVOIDs resolve to the `vs:DataResource` record (e.g. `ivo://org.rubinobs/lsst-dp1/datasets`), while catalog entry IVOIDs resolve to the `vs:CatalogResource` record (`ivo://org.rubinobs/lsst-dp1/catalogs`).


## References

```{bibliography}
  :style: lsst_aa
```
