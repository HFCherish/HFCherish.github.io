---
title: 'MSSQL: multiple cascade paths'
toc: true
tags:
  - Database
date: 2020-05-27 11:10:25
---


# Symptoms

You may receive the following error message when you create a FOREIGN KEY constraint: ([microsoft report](https://support.microsoft.com/en-us/help/321843/error-message-1785-occurs-when-you-create-a-foreign-key-constraint-tha))

```
Server: Msg 1785, Level 16, State 1, Line 1 Introducing FOREIGN KEY constraint 'fk_two' on table 'table2' may cause cycles or multiple cascade paths. Specify ON DELETE NO ACTION or ON UPDATE NO ACTION, or modify other FOREIGN KEY constraints. Server: Msg 1750, Level 16, State 1, Line 1 Could not create constraint. See previous errors.
```

For example, the table definition is like this:

```yaml
Table t1:
	Id: primaryKey

Table t2:
	Id: primaryKey
	parent: ForeignKey(t1, Id) on cascade delete
	child: ForeignKey(t1, Id) on cascade delete
# this would raise the above exception
```



# Cause

Basically, you can't create multiple cascade paths to same table with cascade delete/update. Since you may define `t2.parent` with cascade delete, `t2.child` with cascade update(e.g. update as null), this will make sql server ambiguous when t1 is deleted. MSSQL doesn't detect whether there're actual circle or not, it just forbids the operation to make the design simple.

> A table cannot appear more than one time in a list of all the cascading referential actions that are started by either a DELETE or an UPDATE statement. For example, the tree of cascading referential actions must only have one path to a particular table on the cascading referential actions tree.

[stackoverflow](https://stackoverflow.com/questions/851625/foreign-key-constraint-may-cause-cycles-or-multiple-cascade-paths)

>SQL Server does simple counting of cascade paths and, rather than trying to work out whether any cycles actually exist, it assumes the worst and refuses to create the referential actions (CASCADE): you can and should still create the constraints without the referential actions. If you can't alter your design (or doing so would compromise things) then you should consider using triggers as a last resort.
>
>FWIW resolving cascade paths is a complex problem. Other SQL products will simply ignore the problem and allow you to create cycles, in which case it will be a race to see which will overwrite the value last, probably to the ignorance of the designer (e.g. ACE/Jet does this). I understand some SQL products will attempt to resolve simple cases. Fact remains, SQL Server doesn't even try, plays it ultra safe by disallowing more than one path and at least it tells you so.
>Microsoft themselves [advises](https://support.microsoft.com/en-us/help/321843/error-message-1785-occurs-when-you-create-a-foreign-key-constraint-tha) the use of triggers instead of FK constraints.

# Workaround

Use trigger instead of ForeignKey to keep the integrity and avoid the exceptions.

1. Set cascade delete on t2.child
2. Use `instead of` trigger to cascade delete on t2.parent (you can also use trigger for all fields instead of foreign key constraints)

```
CREATE TRIGGER [DELETE_t2]
   ON dbo.[t1]
   INSTEAD OF DELETE
AS 
BEGIN
 SET NOCOUNT ON;
 DELETE FROM [t2] WHERE parent IN (SELECT Id FROM DELETED)
 DELETE FROM [t1] WHERE Id IN (SELECT Id FROM DELETED)
END
```

