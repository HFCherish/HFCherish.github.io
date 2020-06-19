---
title: 'NHibernate: inverse, cascade'
toc: true
tags:
  - database
date: 2020-05-25 11:38:17
---


[playing-nhibernate-inverse-and-cascade](https://dzone.com/articles/playing-nhibernate-inverse-and),

[nhibernate-inverse](https://web.archive.org/web/20080415101633/http://simoes.org/docs/hibernate-2.1/155.html)

[bidirectional associations](https://nhibernate.info/doc/nhibernate-reference/collections.html#collections-bidirectional)

In database, there may be biodirectional relationships, e.g. Parent has multiple child, and Child has a parent. 
```yaml
#### class definition
class Parent:
	- String id
	- IList<Child> childs	
class Child:
	- String id
	- Parent parent
	
#### db definition
table Parent:
	- id
table Child:
	- id
	- parentId
```

## Inverse

`Inverse` focus on the association. It defines which side is responsible of the association maintenance (create, update, delete), that is, the assignment of `parentId` column. It doesn't care about the maintenance of associated objects which is what `cascade` cares).

By default, `invert=false`, then the assignment of `parentId` is maintened when parent is created/updated/deleted. If we set `parent.child.inverse=true` and the `child.parent is not-null`, then the assignment of `parentId` is maintened when child is created/updated/deleted. 

> `many-to-one` is **always** `inverse="false"` (the attribute does not exist), that is, it means nothing to set `child.parent.inverse=true`


## Cascade

`cascade` instead will focus on the associated objects. It defines if the current object is responsible of the maintenance of associated objects.

By default, `cascade=None`, that is, when saving parent, the childs on parent won't be saved cascadelly.



See [cascade stackoverflow](https://stackoverflow.com/questions/1994433/nhibernate-cascade)

> - none - do not do any cascades, let users handle them by themselves.
> - save-update - when the object is saved/updated, check the associations and save/update any object that requires it (including save/update the associations in many-to-many scenario).
> - delete - when the object is deleted, delete all the objects in the association.
> - delete-orphan - when the object is deleted, delete all the objects in the association. In addition, when an object is removed from the association and not associated with another object (orphaned), also delete it.
> - all - when an object is save/update/delete, check the associations and save/update/delete all the objects found.
> - all-delete-orphan - when an object is save/update/delete, check the associations and save/update/delete all the objects found. In additional to that, when an object is removed from the association and not associated with another object (orphaned), also delete it.