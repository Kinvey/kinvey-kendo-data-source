# Kinvey Kendo UI DataSource

The Kinvey flavor of the [Kendo UI DataSource](http://docs.telerik.com/kendo-ui/api/framework/datasource) component (named `kinvey`) allows you to bind to data stored in your Kinvey apps' collections. It supports filtering, sorting, paging, and CRUD operations.

Content at a glance:

- [Sourcing the Component](#sourcing-the-component)
- [Adding References](#adding-references)
- [Initializing](#initializating)
- [Filtering](#filtering)
- [Sorting](#sorting)
- [Paging](#paging)
- [Unsupported Options](#unsupported-configuration-options) 

## Sourcing the Component

You can choose between several ways of sourcing the Kinvey flavor of Kendo UI DataSoruce, including downloading a local copy or directly referencing an online copy.

### <!--Online Copy-->

<!--For easy setup, you can directly reference the SDK from a Content Delivery Network (CDN).
???Instructions?
-->


> <!--For production apps, we recommend that you install a local copy of the package inside your application. Doing so ensures that the SDK will instantiate even without a network connection.-->

### GitHub Repository

The source code of the component can be accessed in the following [GitHub repository](https://github.com/Kinvey/kinvey-kendo-data-source). You can download a ZIP archive containing the source code from the [Releases](https://github.com/Kinvey/kinvey-kendo-data-source/releases) folder.

## Adding References

It is important to reference the Kinvey flavor of Kendo UI DataSource in your code after referencing Kendo UI and the Kinvey JavaScript SDK flavor that you chose.

```
<!-- Other framework scripts (cordova.js, etc.)  -->

<!-- Kendo UI Core / Kendo UI Professional -->
<script src="kendo.all.min.js"></script>

<!-- Kinvey JS SDK (HTML, PhoneGap, etc.) -->
<script src="...."></script>

<!-- Kinvey Kendo UI Data Source -->
<script src="kendo.data.kinvey.js"></script>
```

## Initializing

To use the `kinvey` data source dialect, [initialize the global `Kinvey` object](https://devcenter.kinvey.com/phonegap/guides/getting-started) first. You can connect the Kinvey flavor of Kendo UI DataSoruce to the backend either by specifying a collection name or by passing an instance of the Kinvey data store object.

> Before initializing the Kendo UI Data Source instance, ensure you have reviewed the [Kinvey Data Store Guide](https://devcenter.kinvey.com/phonegap/guides/datastore) and that you are familiar with how the DataStore types in Kinvey work.

### Initialize with Collection Name

You can connect a Kendo UI Data Source instance directly to a given Kinvey collection by supplying the name of that collection to the `transport.typeName` setting.

When initialized in that way, the Kendo UI Data Source uses a Kinvey Data Store of type `Network` internally.

```javascript
var collectionName = "books";

var dataSourceOptions = {
    type: "kinvey",
    transport: {
        typeName: collectionName
    },
    schema: {
        model: {
            id: "_id"
        }
    },
    error: function(err) {
        alert(JSON.stringify(err));
    }
};

var dataSource = new kendo.data.DataSource(dataSourceOptions);
```

### Initialize with a Data Store Instance

For advanced scenarios like data caching or offline data synchronization you need to handle a Kinvey data store of type `Cache` or `Sync`. You can explicitly instruct the Kendo UI DataSource instance to work with a Kinvey data store instance of any of these types.

When initializing Kendo UI DataSource in this way, you need to manage yourself the state of the store and to `push`, `pull`, and `sync` the entities to/from the server in a suitable place of your application. Kendo UI DataSource only fetches items that are available locally on the device.

```javascript
// initialize the Kinvey data store
var booksSyncStore = Kinvey.DataStore.collection(collectionName, Kinvey.DataStoreType.Sync);
 
// pull and push items to/from the server in a suitable place in your code

// initialize DataSource with the dataStore option
var dataSourceOptions = {
    type: "kinvey",
    transport: {
        dataStore: booksSyncStore
    },
    schema: {
        model: {
            id: "_id"
        }
    },
    error: function(err) {
        alert(JSON.stringify(err));
    }
};

var dataSource = new kendo.data.DataSource(dataSourceOptions);
```

> With a Kinvey data store of type `Cache`, the Kendo UI DataSource instance calls the server directly without consulting the local cache.

## Filtering

Filtering is enabled setting the `serverFiltering` configuration option to `true` and passing a filter object to the `filter` option.

```javascript
var dataSourceOptions ={
    type: 'kinvey',
    // ommitted for brevity
    serverFiltering: true,
    filter: { field: 'author', operator: 'eq', value: 'Lee' }
}
```

The Kinvey dialect supports a selected subset of the Kendo UI DataSource [filter operators](http://docs.telerik.com/kendo-ui/api/framework/datasource#configuration-filter), namely:

- `eq`
- `neq`
- `isnull`
- `isnotnull`
- `lt`
- `gt`
- `lte`
- `gte`
- `startswith` (case-sensitive only)
- `endswith` (case-sensitive only)
- `contains` (case-sensitive only)

In addition to the standard Kendo UI Data Source filtering operators, the Kinvey dialect adds support for the following Kinvey-specific operators:

- `isin`&mdash;value is an array of possible matches.
- `isnotin`&mdash;an inversion of the `isin` logic returning all matches that are not in the specified array, including those entities that *do not* contain the specified field.

```javascript
{
    field: "author",
    operator: "isin",
    value: ["John Steinbeck", "Leo Tolstoy", "Gore Vidal"]
}
```

## Sorting

Sorting is enabled by setting the `serverSorting` configuration option to `true` and passing a sort object to the `sort` option. The Kinvey dialect supports all Kendo UI DataSource [sorting configuration options](http://docs.telerik.com/kendo-ui/api/framework/datasource#configuration-serverSorting).

```javascript
var dataSourceOptions = {
    type: 'kinvey',
    // ommitted for brevity
    serverSorting: true,
    sort: { field: 'author', dir: 'asc' }
}
```

## Paging

Paging is enabled by passing `true` to the `serverPaging` configuration option. 

Setting the `pageSize` configuration option is sufficient for most uses. You can also set the `page` option to request a specific page. The Kinvey dialect supports all DataSource paging [configuration options](http://docs.telerik.com/kendo-ui/api/framework/datasource#configuration-serverPaging).

```javascript
var dataSourceOptions = {
    type: 'kinvey',
    // ommitted for brevity
    serverPaging: true,
    pageSize: 20
};
```

> When `serverPaging` is enabled, a separate request to determine the count of the entities on the server is made *before each request* for the respective page of entities.

### Unsupported Configuration and Operations

The following configuration options of the `DataSource` component are not supported for server execution by Kinvey.

- Standard Kendo UI DataSource filter operators for server filtering:
   - `doesnotcontain`
   - `isempty`
   - `isnotempty`
- Specifying a subset of fields to be returned.
- Sending additional headers with the request.
- [`schema`](https://docs.telerik.com/kendo-ui/api/javascript/data/datasource/configuration/schema)&mdash;`schema.aggregates` and `schema.groups` are not supported. The component already takes care of `schema.data`, `schema.total`, and `schema.type` for you so you do not need to set them explicitly.
- [`batch`](http://docs.telerik.com/kendo-ui/api/framework/datasource#configuration-batch)
- [`serverGrouping`](http://docs.telerik.com/kendo-ui/api/framework/datasource#configuration-serverGrouping)&mdash;you can use client-side grouping instead.
- [`serverAggregates`](https://docs.telerik.com/kendo-ui/api/javascript/data/datasource/configuration/serveraggregates)&mdash;&mdash;you can use client-side aggregation instead or request the data from the [`_group`](https://devcenter.kinvey.com/rest/guides/datastore#aggregation) Kinvey endpoint.
- [`transport.parameterMap`](http://docs.telerik.com/kendo-ui/api/javascript/data/datasource#configuration-transport.parameterMap)
- [`inPlaceSort`](https://docs.telerik.com/kendo-ui/api/javascript/data/datasource/configuration/inplacesort)&mdash;use the `serverSorting` option instead.
- [`offlineStorage`](https://docs.telerik.com/kendo-ui/api/javascript/data/datasource/configuration/offlinestorage)&mdash;instead initialize and work with a Kinvey DataStore of type `SYNC`. 
- [`type`](https://docs.telerik.com/kendo-ui/api/javascript/data/datasource/configuration/type)&mdash;only the `kinvey` value is supported.

### License

See [LICENSE](LICENSE.md) for details.

