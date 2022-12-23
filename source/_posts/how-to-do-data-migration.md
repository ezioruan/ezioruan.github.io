---
title: How to do data migration
date: 2022-07-18 8:18:15
tags: System Architecture
---

#What is Data Migration?
Data migration is the process of moving data from one system to another. While this might seem straightforward, it involves storage, database, or application change.
Any data migration will involve at least the transform and load steps in the context of the extract/transform/load (ETL) process. This means that extracted data needs to go through a series of functions in preparation, after which it can be loaded into a target location.

A strategic data migration plan should include consideration of these critical factors:

- *Knowing the data* — Before migration, source data must undergo a complete audit. Unexpected issues can surface if this step is ignored.
- *Cleanup* — Once you identify any issues with your source data, they must be resolved. This may require additional software tools and third-party resources because of the scale of the work.
- *Maintenance* and protection — Data undergoes degradation after some time, making it unreliable. This means there must be controls in place to maintain data quality.
- *Governance* — Tracking and reporting on data quality is crucial because it enables a better understanding of data integrity. The processes and tools used to produce this information should be highly usable and automate functions where possible.


<!--more-->

A Migration Script must follow:

- *Idempotent*:  means that you can run the migration multiple times, and the end state of the structure will always be the same.
- *Smallest unit*: try to keep the unit as small as possible, and the script must provide extra arguments to run a single unit for testing. A Unit’s operation is in a single transaction, and the failure won’t break the whole process.
- *Migration Log*: all the processes must be logged, either in a log file or a table

A code Example for python Django

the migration script can be run by 
```
# for single
django manage.py migrate_item --item_id=1
# for all
 django manage.py migrate_item --item_id=1
```


the code can be sturctured like below:
```
def migrate_item(item_id):
  //load item from source by id
  source_data = load(item_id)
  // convert to destination format
  destination_data = convert(source_data)
  // get the result and log , the upsert can be an db operation or an api Call
  result = upsert(destination_data)
  log = MigrateLog(id=item_id,source_data=source_data,
    destination_data=destination_data,
    status=result.status,
    error=status.error)
  log.save()


  def migrate(item_id=None):
    if item_id:
      migrate_item(item_id)
    else:
      items = load_items_from_source()
      for item in items:
        migrate_item(item.id)

```
