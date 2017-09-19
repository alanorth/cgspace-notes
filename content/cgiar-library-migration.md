+++
title = "CGIAR Library Migration"
date = 2017-09-18T16:38:35+03:00
description = "Notes on the migration of the CGIAR Library to CGSpace"
categories = ["Notes"]
slug = "cgiar-library-migration"
+++

_Note: I'm temporarily making this a page because it seems Hugo (currently 0.27.1) cannot use a custom slug for a post when there is a permalink defined in `config.toml`_

Rough notes for importing the CGIAR Library content. It was decided that this content would go to a new top-level community called _CGIAR System Organization_.

## Pre-migration Technical TODOs
Things that need to happen before the migration:

- [x] Create top-level community on CGSpace to hold the CGIAR Library content: `10568/83389`
  - [x] Update nginx redirects in ansible templates
  - [x] Update handle in DSpace XMLUI config
- Set up nginx redirects for URLs like:
  - [x] https://library.cgiar.org/bitstream/handle/10947/2699/CGIAR_Branding_Guidelines_and_Toolkit.pdf
  - [x] https://library.cgiar.org/handle/10947/4258
- [x] Merge [#339](https://github.com/ilri/DSpace/pull/339) to `5_x-prod` branch and rebuild DSpace
- [x] Increase `max_connections` in `/etc/postgresql/9.5/main/postgresql.conf` by ~10
  - `SELECT * FROM pg_stat_activity;` seems to show ~6 extra connections used by the command line tools during import
- [x] Temporarily disable nightly `index-discovery` cron job because the import process will be taking place during some of this time and I don't want them to be competing to update the Solr index
- [x] Copy HTTPS certificate key pair from CGIAR Library server's Tomcat keystore:

```
$ keytool -list -keystore tomcat.keystore
$ keytool -importkeystore -srckeystore tomcat.keystore -destkeystore library.cgiar.org.p12 -deststoretype PKCS12 -srcalias tomcat
$ openssl pkcs12 -in library.cgiar.org.p12 -nokeys -out library.cgiar.org.crt.pem
$ openssl pkcs12 -in library.cgiar.org.p12 -nodes -nocerts -out library.cgiar.org.key.pem
$ wget https://certs.godaddy.com/repository/gdroot-g2.crt https://certs.godaddy.com/repository/gdig2.crt.pem
$ cat library.cgiar.org.crt.pem gdig2.crt.pem > library.cgiar.org-chained.pem
```

## Migration Process

**Export all top-level communities and collections from DSpace Test:**

```
$ export PATH=$PATH:/home/dspacetest.cgiar.org/bin
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2515 10947-2515/10947-2515.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2516 10947-2516/10947-2516.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2517 10947-2517/10947-2517.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2518 10947-2518/10947-2518.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2519 10947-2519/10947-2519.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2708 10947-2708/10947-2708.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2526 10947-2526/10947-2526.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2871 10947-2871/10947-2871.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/2527 10947-2527/10947-2527.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10568/93759 10568-93759/10568-93759.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10568/93760 10568-93760/10568-93760.zip
$ dspace packager -d -a -t AIP -e aorth@mjanja.ch -i 10947/1 10947-1/10947-1.zip
```

**Import to CGSpace (also see [notes from 2017-05-10](http://alanorth.github.io/cgspace-notes/2017-05/#2017-05-10)):**

- [x] Copy all exports from DSpace Test
- [x] Add ingestion overrides to `dspace.cfg` before import:

```
mets.dspaceAIP.ingest.crosswalk.METSRIGHTS = NIL
mets.dspaceAIP.ingest.crosswalk.DSPACE-ROLES = NIL
```

- [x] Import communities and collections, paying attention to options to skip missing parents and ignore handles:

```
$ export JAVA_OPTS="-Dfile.encoding=UTF-8 -Xmx3072m -XX:-UseGCOverheadLimit -XX:+TieredCompilation -XX:TieredStopAtLevel=1"
$ export PATH=$PATH:/home/cgspace.cgiar.org/bin
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2515/10947-2515.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2516/10947-2516.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2517/10947-2517.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2518/10947-2518.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2519/10947-2519.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2708/10947-2708.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2526/10947-2526.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-2871/10947-2871.zip
$ dspace packager -r -u -a -t AIP -o skipIfParentMissing=true -e aorth@mjanja.ch -p 10568/83389 10947-4467/10947-4467.zip
$ dspace packager -s -u -t AIP -o ignoreHandle=false -e aorth@mjanja.ch -p 10568/83389 10947-2527/10947-2527.zip
$ for item in 10947-2527/ITEM@10947-*; do dspace packager -r -f -u -t AIP -e aorth@mjanja.ch $item; done
$ dspace packager -s -t AIP -o ignoreHandle=false -e aorth@mjanja.ch -p 10568/83389 10947-1/10947-1.zip
$ for collection in 10947-1/COLLECTION@10947-*; do dspace packager -s -o ignoreHandle=false -t AIP -e aorth@mjanja.ch -p 10947/1 $collection; done
$ for item in 10947-1/ITEM@10947-*; do dspace packager -r -f -u -t AIP -e aorth@mjanja.ch $item; done
```

This submits AIP hierarchies recursively (-r) and suppresses errors when an item's parent collection hasn't been created yet—for example, if the item is mapped. The large historic archive (10947/1) is created in several steps because it requires a lot of memory and often crashes.

**Create new subcommunities and collections for content we reorganized into new hierarchies from the original:**

- [x] Create _CGIAR System Management Board_ sub-community: `10568/83536`
  - [x] Content from _CGIAR System Management Board documents_ collection (`10947/4561`) goes here
  - Import collection hierarchy first and then the items:

```
$ dspace packager -r -t AIP -o ignoreHandle=false -e aorth@mjanja.ch -p 10568/83536 10568-93760/COLLECTION@10947-4651.zip
$ for item in 10568-93760/ITEM@10947-465*; do dspace packager -r -f -u -t AIP -e aorth@mjanja.ch $item; done
```

- [x] Create _CGIAR System Management Office_ sub-community: `10568/83537`
  - [x] Create _CGIAR System Management Office documents_ collection: `10568/83538`
  - Import items to collection individually in replace mode (-r) while explicitly preserving handles and ignoring parents:

```
$ for item in 10568-93759/ITEM@10947-46*; do dspace packager -r -t AIP -o ignoreHandle=false -o ignoreParent=true -e aorth@mjanja.ch -p 10568/83538 $item; done
```

**Get the handles for the last few items from CGIAR Library that were created since we did the migration to DSpace Test in May:**

```
dspace=# select handle from item, handle where handle.resource_id = item.item_id AND item.item_id in (select item_id from metadatavalue where metadata_field_id=11 and date(text_value) > '2017-05-01T00:00:00Z');
```

- Export them from the CGIAR Library:

```
# for handle in 10947/4658 10947/4659 10947/4660 10947/4661 10947/4665 10947/4664 10947/4666 10947/4669; do /usr/local/dspace/bin/dspace packager -d -a -t AIP -e m.marus@cgiar.org -i $handle ${handle}.zip; done
```

- Import on CGSpace:

```
$ for item in 10947-latest/*.zip; do dspace packager -r -u -t AIP -e aorth@mjanja.ch $item; done
```

## Post Migration

- [x] Shut down Tomcat and run `update-sequences.sql` as the system's `postgres` user
- [x] Remove ingestion overrides from `dspace.cfg`
- [x] Reset PostgreSQL `max_connections` to 183
- [x] Enable nightly `index-discovery` cron job
- [x] Adjust CGSpace's `handle-server/config.dct` to add the new prefix alongside our existing 10568, ie:

```
"server_admins" = (
"300:0.NA/10568"
"300:0.NA/10947"
)

"replication_admins" = (
"300:0.NA/10568"
"300:0.NA/10947"
)

"backup_admins" = (
"300:0.NA/10568"
"300:0.NA/10947"
)
```

I had been regenerated the `sitebndl.zip` file on the CGIAR Library server and sent it to the Handle.net admins but they said that there were mismatches between the public and private keys, which I suspect is due to `make-handle-config` not being very flexible. After discussing our scenario with the Handle.net admins they said we actually don't need to send an updated `sitebndl.zip` for this type of change, and the above `config.dct` edits are all that is required. I guess they just did something on their end by setting the authoritative IP address for the 10947 prefix to be the same as ours...

- [ ] Update DNS records:
  - CNAME: cgspace.cgiar.org
- [x] Re-deploy DSpace from freshly built `5_x-prod` branch
- [x] Merge `cgiar-library` branch to `master` and re-run ansible nginx templates
- [x] Run system updates and reboot server
- [ ] Switch to Let's Encrypt HTTPS certificates (after DNS is updated and server isn't busy):

```
$ sudo systemctl stop tomcat7
$ ./letsencrypt-auto certonly --standalone -d library.cgiar.org
```

## Troubleshooting

### Foreign Key Error in `dspace cleanup`
The cleanup script is sometimes used during import processes to clean the database and assetstore after failed AIP imports. If you see the following error with `dspace cleanup -v`:

```
Error: ERROR: update or delete on table "bitstream" violates foreign key constraint "bundle_primary_bitstream_id_fkey" on table "bundle"                                                                                                                       
  Detail: Key (bitstream_id)=(119841) is still referenced from table "bundle".
```

The solution is to set the `primary_bitstream_id` to NULL in PostgreSQL:

```
dspace=# update bundle set primary_bitstream_id=NULL where primary_bitstream_id in (119841);
```

### PSQLException During AIP Ingest
After a few rounds of ingesting—possibly with failures—you might end up with inconsistent IDs in the database. In this case, during AIP ingest of a single collection in submit mode (-s):

```
org.dspace.content.packager.PackageValidationException: Exception while ingesting 10947-2527/10947-2527.zip, Reason: org.postgresql.util.PSQLException: ERROR: duplicate key value violates unique constraint "handle_pkey"                                    
  Detail: Key (handle_id)=(86227) already exists.
```

The normal solution is to run the `update-sequences.sql` script (with Tomcat shut down) but it doesn't seem to work in this case. Finding the maximum `handle_id` and manually updating the sequence seems to work:

```
dspace=# select * from handle where handle_id=(select max(handle_id) from handle);
dspace=# select setval('handle_seq',86873);
```
