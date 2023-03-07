# Entity States

How does JPA know when to write data from an object to a table, or when an object is no longer valid?

JPA does this by managing its entities in 4 states.

1. Transient (New)
2. Persistent (Managed)
3. Detached (Unmanaged)
4. Removed (Deleted)

## 1. Transient

An object that is newly created and has never been associated with JPA Persistence Context is considered to be in the "Transient" state.
The properties set in this object is not stored in the database.
It is a new object and will usually be moved to the "persistent" state by the entity manager.

## 2. Persistent

An instance of an entity (or simply a object) in the "persistent" state represents a record in the table.

## 3. Detached

An Object becomes detached when the currently running Persistence Context is closed.
Any changes made to detached objects are no longer automatically propagated to the database.

## Reference

https://jstobigdata.com/jpa/different-states-of-an-object-in-jpa

https://javarevisited.blogspot.com/2017/04/difference-between-transient-persistent-and-detached-state-hibernate-java.html
