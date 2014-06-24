# Appbase Overview

## Objects and Properties

An _object_ in Appbase very similar to a JSON object, and contains key/value pairs. The keys in an object are called the _properties_ of the _object_. An object can be created using `Appbase.new({pk: key})`, where the `key` is a _primary key_ given to the object, and this key can be used later on to _fetch_ the object from Appbase. If no `key` is given, a random string will be assigned as the key for the object. `Appbase.new()` returns an `Appbase Reference Object` which is a reference to the object stored in Appbase. Read/write operations can be carried out this reference object.

Properties can be accessed using __TODO__ method and they can only contain primitive data (Number/String). To set value for a property, use __TODO__ method.

Code Example:
```javascript
TODO
```

## Namespaces

Every object belongs to a _namespace_. Namespaces can be used to categorize the _types_ of objects. For eg., in a chat application, the namespaces could be: `User, Chat, ChatGroup` etc. 

Namespace are __NOT__ containers of objects, and the _primary key_ of an object is unique across all the namespaces. Namespaces are there to categorize objects. This categorization helps applying security rules on objects which are alike, and should be treated much the same. For eg., the objects of the `ChatGroup`, must only be visible to the members of the chat group. Such a rule can be imposed on the namespace and all the objects in that namespace would follow the rule. 

`Appbase.new({ns: namespace, pk: key})` method creates objects in the given `namespace`, and if no namespace is specified, the objects goto into the namespace called __Default__. 

Namespaces are automatically created when a new namespace is specified in `Appbase.new()`, although this behavior can be controlled by security rules. An object can be moved amongst namespaces, which might be useful over the course of development. 

To access an object which was previously created using `Appbase.new()`, use `Appbase.ref(app_url/namespace/key)`, which returns an Appbase Reference to the object.

Code Example:
```javascript
TODO
```

## The Graph

Today's application are complex and objects in the application are linked to eachother in a number of ways and dimensions. For eg. social applications. The data of such applications can not be modeled in a simple Table-Row or Tree data structure.

### Links

In _Appbase_ a number of links can be pointed from one object to other objects, allowing you to create a _Graph_. 

Links in an object are _named_. This allows having _one to one_ relationships amongst objects. For eg. In a chat application, every chat would be posted by a user, and this relation  can be modeled as a named link in Appbase. Other such examples are given below.

\#|From|Relation|To|
-|-|-|-|
1|`Chat object`| posted_by |`User Object`|
2|`Document object`|owner|`User object`|
3|`User object`|subscribes_to|`User/Channel object`|

Code Example:
```javascript
TODO
```

For convenience, if no name is given to the link, _primary key_ of the object being added in the link, becomes the link name. This way, an object can be used as a container of other objects.

Code Example:
```javascript
TODO
```

Objects as containers are also useful for creating _one to many_ relationships. Look at the 3rd example in the table given above. A user may subscribe to number of channels. In this case, `User` object's link `subscribes_to` could point to a `ChannelsContainer` object, which can have any number of channels.

Code Example:
```javascript
TODO
```

Links in an object are automatically arranged in the increasing order starting from `0`, i.e. the first link being added will get the index `0`, the second would get `1` and so on. While retrieving linked objects, one can specify the starting and ending indexes. The order of a link can be specified manually while adding the link, or it can be modified later on.

Code Example:
```javascript
TODO
```

### Paths

A _path_ in _Appbase_, is a way to access the graph. 

It starts from the url given to application, followed by a namespace and the primary key of an object. Eg. `https://shawshank.api.appbase.io/user/andy_dufresne` points to th `user` object with primary key `andy_dufresne`. 

This object would be the entry point, and the graph can be explored via object's links. Eg. `https://shawshank.api.appbase.io/user/andy_dufresne/best_friend` points to the object added as the link `best_friend` in the `user` object with primary key `andy_dufresne`. This path can be given to `Appbase.ref(path)` and the returned _Appbase Reference Object_ will point to the linked object and read/write operations on this reference, will affect the linked object. 

Code Example:
```javascript
TODO
```

## Realtime and Sync

### Events
The graph, as it keeps changing, these changes can be listened in realtime and if you missed some changes while being offline, they will be automatically pulled from where you left and local changes will be automatically pushed. Checkout the API Docs __TODO: Link__, to know how conflicts can be handled.

In _Appbase_, there are five kind of events fired on an object: 

\#|Event|Fired when|
-|-|-|
1|`value`| Object's data is changed, ie. a property is added, removed, or its value is changed
2|`link_added`| A new link is added
3|`link_removed`| A link is removed 
4|`link_changed`| A named link now points to a different object, or its order is manually changed. Note: It has nothing to do with changes in the data of the linked object - __TODO: A Discussion__
5|`link_value`| A link's data is changed. Note: Use only when critically needed, as in the background this event listens to `value` event for _all_ the links - __TODO: A Discussion__

Whenever an event is fired, the callback is passed an _Appbase Snapshot Object_, which includes the stored data and ordering details. Take a look at the API Docs __TODO: Link__ for more details on _Snapshot Object_. 

### Reading Data from Appbase

In _Appbase_, the only way to read data is listening to events. `value` event returns  _existing_ data at first, and later on, is fired whenever the data is changed. 

Similarly,  `link_added` event returns _existing_ links at first, and is subsequently fired when a new link is added. Although, this behavior can be controlled via `limit` and `startAt` options, take a look at the API Docs __TODO: Link__. 

Events `link_removed`, `link_changed`, `link_value` are fired only when the data is _modified_.

Code Example:
```javascript
TODO
```

### How the events are fired

__Not sure whether to keep this section here. It helps understanding Appbase, and clarifies some edge cases.__

Appbase client library _caches_ the data locally, in a _graph_ like data structure. In the background it uses the REST api __TODO: Link__, and listens to the paths requested by the user. When new data is arrived, it compares the data with the cache, and the events are fired accordingly. 

Eg. While listening to link events, if the data arriving data is `{name:'subscribes_to', points_to: 'ABCXYZ', order:5}`. Now,

1. If the link named `subscribes_to` exists in the cache, `link_changed` event is fired.
2. If the link doesn't exist, `link_added` event is fired.

If the data arriving data is `{name:'subscribes_to', points_to: null}`. Then,

1. If the link named `subscribes_to` exists in the cache, `link_removed` event is fired.
2. If the link doesn't exist, the data will be ignored and no event is fired.
