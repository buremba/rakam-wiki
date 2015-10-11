## Collecting Events <sub>*[api-doc](//getrakam.com/api/tags=event,collection)*</sub>
Rakam provides client libraries for a few programming languages for collecting events easily.
Also you can send events Rakam in JSON format to an endpoint using a RESTFul Web Service that Rakam provides. 

Client APIs:
- [Javascript Client Library]()
- [Java Client Library]()
- [Python Client Library]()
- [Sending event data in JSON data]()

> You can also send event data in JSON to the RESTFul web service of Rakam but we suggest you > to use Rakam client libraries if it's available. You can also find the specification of the > client APIs if you want to develop a client library for Rakam. [Specification]()

You can think of event as the row of a table in RDBMSs. Similar to the tables in RDBMSs, we have event collections that store the events. In order to use Rakam, you need to create project with a unique name and send event to that project. The events are stored in event collections inside a project. Event collections also have schemas but we don't enforce you do define the schema before collecting events. We can generate the schema in runtime automatically so that you don't have to do that manually.

We have aimed to make the schema evolution easy since the analytical systems almost always need it and it can be hard to do manually in runtime. As you start to send your events to Rakam, it will create the schemas based on the value types of the fields that you send as event properties. Let's say you send the following event to the Rakam:

```javascript
{
    "collection": "pageView",
    "properties": {
        "user_agent": "Firefox",
        "locale": "TR-tr",
        "url": "http://mysite.com/blog-post",
        "referrer": "http://google.com/?q=term",
        "ip": "186.45.356.33",
        "platform": "web",
        "page_duration": 5,
        "session_id": "c88d7bad-af01-433f-bef1-dc107cee4334"
        "user": "fdsy8fy7d"
    }
}
```
Rakam will generate the schema for this event collection based on the values that you sent. Then, creates the collection in backend storage. For example, Postgresql backend will execute a CREATE TABLE query similar to this one:

```sql
CREATE TABLE pageView (
    user_agent     TEXT,
    locale         TEXT,
    url            TEXT,
    referrer       TEXT,
    ip             TEXT,
    platform       TEXT,
    page_duration  BIGINT,
    session_id     TEXT,
    user           TEXT
);
```

Similarly, when Rakam encounters a new field, it will update the table with the new field. Therefore, even though Rakam has the ability to update the schema, it's always a good practice to have a fixed set of fields because the events are usually not stored as JSON data in underlying database, they usually have a strict schema.

> Rakam will check the fields and if they exists and values match the existing schema, the event will be sent to the storage backend as you sent. However; let's say the field *ip* is created previously and the type is *integer*. Since *string* can't cast to *integer*, the field *ip* will be ignored.

> The schema evolution feature may cause a security problem if you don't have the control on > the client side that sends the events to Rakam. For example, if you install the Javascript > tracker on website that sends the events directly from the users' browsers to Rakam, an attacker may send randomly generated fields to Rakam so you may end up event collections that have 100s of fields. Therefore we suggest disabling dynamic schemas using *disable_dynamic_schema=true* once you created the schema of your event collections or disable this feature completely on production. See config: [disable_dynamic_schema]().
