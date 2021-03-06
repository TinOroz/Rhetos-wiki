Contents:

1. [Data model and relationships](#data-model-and-relationships)
2. [Simple business rules](#simple-business-rules)
3. [Computed data and read-only data structures](#computed-data-and-read-only-data-structures)
4. [Filters](#filters)
5. [Reports](#reports)
6. [Complex concepts](#complex-concepts)
7. [Server actions](#server-actions)
8. [Claims and permissions](#claims-and-permissions)
9. [Low-level object model concepts](#low-level-object-model-concepts)
10. [Database objects](#database-objects)

## Data model and relationships

[Data model](Data-model-and-relationships):

* **Module** `<name>` - Generates a C# namespace and a database schema.
* **DataStructure** `<Module>.<name>` - Basic concept for any data structure (e.g. Entity, Browse, Computed, ...) which may contain properties. It is usually not directly used in DSL scripts.
* **Entity** `<Module>.<name>` - Represents data entered by a user or generated by the system. Creates a C# class and database table. Inherits the DataStructure concept.

[Simple property types](Data-structure-properties):

* **ShortString** `<DataStructure>.<property name>` - Maximal length 256 characters.
* **LongString** `<DataStructure>.<property name>` - Unlimited length.
* **Integer** `<DataStructure>.<property name>`
* **Decimal** `<DataStructure>.<property name>`
* **Money** `<DataStructure>.<property name>`
* **Bool** `<DataStructure>.<property name>`
* **Date** `<DataStructure>.<property name>`
* **DateTime** `<DataStructure>.<property name>`
* **Guid** `<DataStructure>.<property name>`
* **Binary** `<DataStructure>.<property name>`

[Relationships](Data-model-and-relationships) and [navigation properties](Data-structure-properties):

* **Reference** `<DataStructure>.<property name> <referenced data structure>` - Lookup field (N:1 relationship). The generated C# property has type of the referenced class. Generates foreign key constraint in database. The database column has "ID" suffix.
* **Reference** `<DataStructure>.<property name>` - Simplified syntax in case when the property name is same as the name of the referenced entity.
* **UniqueReference** `<DataStructure>.<property name> <referenced data structure>` - Same as reference but with one-to-one relationship. Use when extended entity should not be a part of the base entity.
* **Detail** `<Reference>` - Marks entity as detail of the referenced entity. Generates Required, SqlIndex and CascadeDelete.
* **Extends** `<extension DataStructure>.<base DataStructure>` - Marks first data structure as an extension of the base data structure (1:1 relationship). Often used to extend entity with additional computed data. The extension records have the same primary key value as the base records.
* **LinkedItems** `<DataStructure>.<property name> <detail Reference>` - A property that contains a list of detail records.
* **CascadeDelete** `<Reference>` - When deleting the referenced record, the related detail records will be automatically deleted.
* **PrerequisiteAllProperties** -  This concept is used as a placeholder for internal optimization when all properties of an entity are required as a prerequisite for another concept.
A dependent object can reference this concept as a dependency, instead of referencing each property individually.

[Browse](Read-only-data-structures):

* **Browse** `<Module>.<name> <original data structure>` - Data structure used for simple data queries, when we only need to select some properties from an entity and other referenced data structures.
* **Take** `<Browse>.<property name or reference path>` - Creates a property on the browse data structure that returns the property value from the base data structure. Instead of a simple property, it is possible to specify a path across multiple references, separated by a dot.
  * The resulting property name is automatically generated by connecting the names from the path (e.g. `Take 'Parent.Organization.Name'` will result with property `ParentOrganizationName`).
* **Take** `<Browse>.<property name> <reference path>` - Explicitly set the name of created property.
* **Snowflake** is obsolete and renamed to `Browse`.

Hardcoded:

* **Hardcoded** `<Module>.<name>` - Data structure that represents a simple read-only entity or a code table.
* **Entry** `<Hardcoded>.<entry name>` - Creates a row in the database for every entry.
* **Value** `<Entry>.<property name> <value>` - Adds the value to the column.

LegacyEntity:

* **LegacyEntity** `<Module>.<name> <table> <view>` - Simplifies access from Rhetos application to legacy application database. It maps a Rhetos data structure to the legacy database table or view. It allows both read and write operations (either with updateable views or generated instead-of triggers). It allows mapping of complex primary and foreign keys to standard Rhetos reference properties. Prerequisites: The legacy table needs to be extended with uniqueidentifier ID column with default `NEWID()` and a unique index.
* **LegacyEntity** `<Module>.<name> <table>` - Creates view with entity name and the corresponding instead-of triggers for writing data.
* **LegacyProperty**
* **LegacyPropertyReadOnly**

Other concepts:

* **Implements** `<DataStructure>.<interface type>` - Implements the given C# interface on the entity's POCO class.
* **PropertyFrom** `<destination DataStructure>.<source Property>` - Copies property from another data structure, along with the associated Required and SqlIndex concepts.
* **AllPropertiesFrom** `<destination DataStructure>.<source DataStructure>` - Copies all properties from another data structure, along with the associated Required, SqlIndex and Extends concepts.
* **AllPropertiesWithCascadeDeleteFrom** `<destination DataStructure>.<source DataStructure>` - Copies all properties from another data structure, along with the associated Required, SqlIndex, Extends and CascadeDelete concepts.
* [**Polymorphic**](Polymorphic-concept) `<Module>.<name>` - A common interface that can be implemented by multiple entities.
  * **Is** `<Entity>` - References the implemented entity
  * **Implements** `<Property> <SQL expression>` - Implement a property by a specific SQL expression
  * **Where** - Is is used to limit the items that will be included in the polymorphic implementation.
  * **Materialized** - Manually creates a materialized table, when there are no references or extensions to the polymorphic
  * **SqlImplementation** - Instead of using property implementations (`Implements keyword`), a specific SQL query may be provided to implement the mapping between the subtype and the polymorphic entity.
  * **SqlDependsOn** - Is a dependency information that is used when creating database objects.

## Simple business rules

See the full article: [Implementing simple business rules](Implementing-simple-business-rules).

Property value constraints:

* **Required** `<Property>` - The property value must be entered when saving a record. There are two subvariant of this concept:
  * **SystemRequired** `<Property>` - The value must be entered by system internally.
  * **UserRequired** `<Property>` - User (or client applications) needs to provide the property value.
  * **RequiredAllowSave** - for suppressing `Required` property validations on save.
  * **SqlNotNull** - This concept is intended for internal use only, for other concepts' implementations. Use `Required` or `SystemRequired` instead to implement business requirements.
* **Unique** - `<Property>` - Two records cannot have same value of this property.
  * **UniqueMultiple** `<DataStructure>.<property names separated by comma>` - A unique constraint over multiple properties: Two records cannot have same combination of values.
* **MinValue** `<Property>` - Limits the smallest allowed value of the property.
* **MaxValue** `<Property>` - Limits the largest allowed value of the property.
* **MinLength** `<Property>` - Limits the string length in ShortString or LongString property.
* **MaxLength** `<Property>` - Limits the string length in ShortString or LongString property.
* **Range** `<PropertyFrom> <PropertyTo>` - Value of the first property has to be less or equal to the second property (properties have to be of the same type).
  * **DateRange** `<PropertyFrom> <PropertyTo>` - Inherits the Range concept and generates a Date property in case it does not exist already.
  * **DateTimeRange** `<PropertyFrom> <PropertyTo>` - Inherits the Range concept and generates the DateTime properties in case they don't exist already.
  * **IntegerRange** `<PropertyFrom> <PropertyTo>` - Inherits the Range concept and generates the Integer properties in case they don't exist already.
* **RegExMatch** `<Property> <pattern> <error message>` - Use a regular expression to validate the string property value.

Deny data modifications:

* [**InvalidData**](Data-validations) `<DataStructure>.<filter name> <message>` - Standard data validation. When saving data, if any record matches the given filter, the operation will be canceled with and the error message returned to the user.
  * **AllowSave** - Modifies the `InvalidData` concept to suppress data validation on save. Instead, the data validation can be excepted separately as a report.
  * **DenySave** - is obsolete use InvalidData instead
  * **MarkProperty** - If the error is related to the single property, use this to add that information to the validation
  * **ErrorMetadata** - Adds any custom metadata information to the error response. This metadata may be used in frontend to display the error message in a certain way.
  * **MessageParametersItem** - Extends the validation's error message with custom parameter values.
  * **MessageParametersConstant** - It has subset of the features of `MessageParametersItem`. It works in a similar way, but it can only provide constant values.
  * **MessageFunction** - Allows full control over validation's error message and metadata. It is implemented as a lambda expression the returns the error message and metadata for a given list of invalid record IDs.
* **Lock** `<DataStructure>.<filter name> <message> <property>` - Deny update and delete of the entity records, for records in a certain state (provided by a filter).
* **LockProperty** `<DataStructure>.<filter name> <message>` - Deny update of a property, for records in a certain state.
* **LockExcept** - Deny update (except for properties from the provided list) and delete of the entity records, for records in a certain state.
* **DenyUserEdit** `<Entity>` - Client application is not allowed to directly insert, update or delete the entity records (no condition).
* **DenyUserEdit** `<Property>` - Client application is not allowed to directly insert or update the property.

Automatically generated data:

* **AutoCode** `<ShortString property>` - Automatically generate numeric codes. The codes can have a given string prefix and a fixed number of digits. It generates Unique constraint on the property.
  * **AutoCodeCached** - An optimized version of `AutoCode` for large tables. It stores the latest used code, so it does not need to read the existing records when generating a new code, but it requires manual initialization the persisted data at initial deployment or import database records.
* **AutoCodeForEach** `<ShortString property> <grouping property>` - Same as `AutoCode`, but the numbers are starting from 1 within each group of records. The group is defined by the second property value.
  * **AutoCodeForEachCached** -  Equivalent to `AutoCodeForEach` with cached last code values.
* **CreationTime** `<DateTime property>` - Automatically enters time when the records was created.
* **CreatedBy** - Writes the current user's ID when saving a new record.
* **ModificationTimeOf** `<DateTime property> <Modified property>` - Automatically enters time when some given property was last updated.
* **DefaultValue** `<lambda>` - for setting the default property values when inserting a new record.

Logging data changes and auditing:

* **Logging** `<entity>` - Creates a database trigger that monitors all inserts, updates and deletes, and writes them to `Common.Log` table.
* **Log** `<Logging <Property>` - Sets properties which changes are tracked.
* **AllProperties** `<Logging>` - Enables logging on all properties.
* **LogReaderAdditionalSource** - Enables implementation of a custom Log archive. The archive can be integrated into the existing auditing features by extending `LogReader` and `LogRelatedItemReader`. A low-level concept that inserts the SQL code snippet to the log reader SqlQueryable at the place of the given tag (an SQL comment).

Other features:

* **Deactivatable** `<Entity>` - Allows tracking of active and deactivated records. Generates property `Bool Active`.
* **PessimisticLocking** `<Entity>` - Enables automatic verification of explicit client locks when saving a record. Generates server actions SetLock and ReleaseLock.

## Computed data and read-only data structures

[Read-only data structures](Read-only-data-structures):

* **Computed** `<Module>.<name> <lambda>` - A readably data structure, implemented with a C# code (the lambda expression) that returns an array of the records: `repository => array of records`.
* **SqlQueryable** `<Module>.<name> <SQL script>` - A queryable data structure, implemented with an SQL query. It generates a database view. With this concept, you should always use SqlDependsOn or AutodetectSqlDependencies to make sure that database objects can be created in the correct order.
* **QueryableExtension** `<Module>.<name> <source DataStructure> <lambda>` - A queryable data structure, implemented with a C# code (the lambda expression), for extending entities with simple computed data. The lambda expression returns records as a query: `(IQueryable<BaseEntity> source, repository) => IQueryable<Extension>`.

Additional concepts:

* **ExternalReference** `<Module>.<type or assembly>` - Adds the dll reference for the generated application. The dll can be references in two ways:
  * (Recommended) By C# type which is used (the assembly qualified name). Version, Culture or PublicKeyToken can be removed from the AssemblyQualifiedName for dlls that are placed in the Rhetos application folder.
  * By dll name (e.g. 'Rhetos.MyFunctions.dll').
* **UseExecutionContext** `<computation>` - Adds an additional input parameter "ExecutionContext" to the Computed or QueryableExtension lambda expression.
  This concept is obsolete, repository member `_executionContext` should be used instead.

[Saving computed data](Persisting-the-computed-data):

* **Persisted** `<Module.<name> <source DataStructure>` - Creates entity which will contain cached data from the given source. The source can be any readable data structure (Browse, SqlQueryable, Computed and similar).
* **ComputedFrom**  is a more flexible version of `Persisted`. It allows a property-level recomputing instead of entity-level. It is intended to be used as an internal concept for building simpler macro concepts.
  * **KeyProperty** and **KeyProperties** - for `ComputedFrom` to control when to update an item or to delete old item and insert a new one.
  * **KeyPropertyID** - ?
  * **SkipRecomputeOnDeploy** - for disabling recompute-on-deploy for a specific entity in the DSL script (issue #105).
* **AllProperties** `<Persisted>` - Copies all properties from source data structure to the cache entity, along with the associated Required, SqlIndex, Extends and CascadeDelete concepts.
* **KeepSynchronized** `<Persisted>` - Automatically updates cache when the source data is changed. This feature requires ChangesOnChangedItems defined on the source.
* **KeepSynchronized** `<Persisted> <save filter>` - Same as above, but leaving some computed records unchanged. The "save filter" is a lambda expression `(IEnumerable<Entity> items, repository) => IEnumerable<Entity>`, that returns subset of items which are allowed to be updated by the KeepSynchronized mechanism.
  * **DefaultLoadFilter** - for limiting automatic computation on a subset of rows on `KeepSynchronized`.
  * **SuppressSynchronization** - Internal concept for turning off automatic recompute of an entity with KeepSynchronized.
* **ComputeForNewBaseItems**
* **ComputeForNewItems** - New persistence control mode

Defining dependencies for the computation source, so that KeepSynchronized can know when to update the cached data:

* **ChangesOnBaseItem** `<ComputationSource>` - If the computation is an extension of a base entity: When a base record is saved, the related cache record should be recomputed.
* **ChangesOnLinkedItems** `<ComputationSource> <ReferenceProperty>` - If the computation is an aggregation of a detail entity: When a detail record is saved, the related parent's cache record should be recomputed.
  * For example, if you are computing additional data about a document and that data depends on a detail entity (for example, the number of items, or the total amount) then the parameter "ReferenceProperty" should be the full name of the reference property from the detail entity.
* **ChangesOnReferenced** `<ComputationSource> <path to referenced entity>` - If the computation depends on a referenced entity: When a referenced entity record is saved, all cache records that reference it should be recomputed. Instead of a simple property, it is possible to specify a path across multiple references, separated by a dot, including the `Base` and `Extension_...` navigation properties.
* **ChangesOnChangedItems** `<ComputationSource> <entity> <filter name> <filter snippet>` - Programmable concept for defining dependency for computed items.
  * The "filter snippet" is a lambda expression that for an array modified dependent items returns filter parameter "what cached items need to be recomputed": `DependentEntity[] changedItems => filter parameter`.
  * The filter has to be applicable to both source and cache data structure. FilterAll, System.Guid[] and FilterCriteria are commonly used filters, supported by all entities.
  * Examples:
    * If *any* change of the dependent entity should result with recomputing *all* cache records, it can be declared this way: `ChangesOnChangedItems Test.Item 'FilterAll' changedItems => new FilterAll()';`
    * ChangesOnBaseItem generates:
      `ChangesOnChangedItems Test.Item 'Guid[]' 'changedItems => changedItems.Select(item => item.ID).ToArray()';`
    * ChangesOnLinkedItems generates:
      `ChangesOnChangedItems Test.Item 'Guid[]' 'changedItems => changedItems.Where(item => item.ParentID != null).Select(item => item.ParentID.Value).Distinct().ToArray()';`
    * ChangesOnReferenced generates something similar to:
      `ChangesOnChangedItems Test.Item 'FilterCriteria' 'changedItems => new FilterCriteria("path to referenced entity", "In", changedItems.Select(item => item.ID))';`
* **KeyProperties** - A list of properties that are used as a key when comparing the data from source to the cache. By default, the data is matched by ID.

## Filters

See the full article: [Developing filters and other read methods](Filters-and-other-read-methods).

* **FilterBy** `<DataStructure>.<parameter type> <lambda>` - A read method that loads and returns data for the given parameter value.
  The lambda expression returns an array of records: `(repository, parameter) => filtered DataStructure[]`.
  The parameter type also represents the filter name.
* **Query** `<DataStructure>.<parameter type> <lambda>` - It is similar to the FilterBy described above, it just returns a query instead of a simple array.
* **ComposableFilterBy** `<DataStructure>.<parameter type> <lambda>` - A read method that returns a filtered query for the given source query and the parameter value.
  The lambda expression returns a subset of a given query: `(IQueryable<DataStructure> query, repository, parameter) => filtered IQueryable<DataStructure>`.
  The parameter type also represents the filter name.
  * **ComposableFilterByReferenced** for inheriting filters from referenced data structure. Use it to additionally filter out some items or sort the items within a group with the same reference value.
* **ItemFilter** `<DataStructure>.<filter name> <lambda>` - Helper concept for simplified definition of simple "one-liner" filters, that generates a ComposableFilterBy.
  The lambda expression returns whether each records passes the filter: `item => bool`.
  * **ItemFilterReferenced** for inheriting filters from referenced data structure. Use it to additionally filter out some items or sort the items within a group with the same reference value.

Additional concepts:

* **Parameter** - Data structure for FilterBy and ComposableFilterBy parameters. It also represents the filter name.
  Although any data structure can be used as filter parameter, the Parameter concept describes an intention behind the data structure.
* **UseExecutionContext** `<FilterBy>` - Adds an additional input parameter "ExecutionContext" to the filter's lambda expression.
  This concept is obsolete, repository member `_executionContext` should be used instead.
* **FilterByReferenced** `<detail DataStructure>.<parameter type> <parent Reference> <lambda>` - Copies a FilterBy filter implementation from the parent data structure to the detail.
  The lambda expression can contain additional data processing of the filtered items: `IEnumerable<Detail> => .. additional filter or sort from group with the same parent`.
* **FilterByLinkedItems** `<parent DataStructure>.<parameter type> <detail Reference>'` -  Copies a FilterBy filter from a detail data structure to the parent.
* **FilterByBase** `<extension DataStructure>.<parameter type>`
* **ApplyFilterOnClientRead** and **ApplyOnClientRead**, for filters that are automatically added to each client request. The given filter will be automatically applied when executing ReadCommand server command (the command is used in SOAP and REST API).

Beside these explicitly defined filters, generic filters are available in [Web API](https://github.com/Rhetos/RestGenerator/blob/master/Readme.md#reading-data)
for all queryable data structures (Entity, Browse, SqlQueryable, QueryableExtension, ...).

## Reports

* **ReportFile** `<Module>.<name> <lambda>` - The lambda expression returns an arbitrary generated file (file name and contents as byte array), based on the report data collected by DataSources.
  The lambda expression returns the file:  `(object[][] reportData, string convertFormat, executionContext) => new Rhetos.Dom.DefaultConcepts.ReportFile {string Name, byte[] Content}`
  * Report is also a data structure. Properties on the report represent report parameters.
  * The report is available in Web API.
* **ReportData** `<Module>.<name>` - A subset of ReportFile features: allows downloading the raw report data, without generating the report file. In the object model, only `GetReportData` method is generated.
* **DataSources** `<Report>.<comma separated data structures>` - List of readable data structures that provide the data for the report. The module name is optional.
  * Each provided data source should have a FilterBy implementation with the filter parameter name same as the report name.
  * It is recommended to use FilterByBase, FilterByReferenced and FilterByLinkedItems, to avoid writing redundant filters on related structures (e.g. report may filtered documents and the related detail items).
* **DataSource** `<Report>.'order' <DataStructure>` - Definition of a single data source for the report. Use the DataSources concept instead, it automatically generates the DataSource statements.
* **ConvertFormat** `<Report>.<format extension>` - Automatically sets the expected report format on download. If set, the user cannot choose the format.
* [**TemplaterReport**](TemplaterReport) `<Module>.<name> <file path>` - A report that fills a predefined template with the data collected by DataSources.
  * Supported templated files: doc, docx, xls and xlsx. Supports downloading report in original format or converter to pdf.
  * The file path should be formed as: `package name\file name`
  * **REMARK:** This concept is not available in CommonConcepts package. It is required a separate "TemplaterReport" DSL package.

## Complex concepts

Hierarchy:

* **Hierarchy** `<DataStructure>.<Reference name>` - Adds a self-reference property (for example, a superior in the employee hierarchy).
  * Generates a cached index for optimized recursive queries (for example, find all direct and indirect subordinates) in the extension `<EntityName><HierarchyName>Hierarchy`.
  * Generates ComposableFilters `<HierarchyName>HierarchyDescendants` and `<HierarchyName>HierarchyAncestors` for quick access of all direct and indirect child records or parents.
  * Generates validations to deny entering data with circular dependencies.
* **Hierarchy** `<DataStructure>.<Reference name> <PathName> <PathProperty> <PathSeparator>` - Same as Hierarchy above, with additionally cached full path for each record.
* **SingleRoot** `<Hierarchy>` - Limits insert to only one root record.

[History](Temporal-data-and-change-history):

* **History** `<Entity>` - Enables temporal data management on the entity:
  Automatically keeps old versions of each records.
  Allows reading record's state at a given point in time.
  Allow entering data values that were effective from a previous point in time.
* **History** `<Property>` - Includes the entity's property in temporal data management.
* **AllProperties** `<History>` - Includes all properties in temporal data management.

## Server actions

See the full article: [Action concept](Action-concept).

* **Action** `<Module>.<name> <lambda>` - A custom server action that executes the code in the given lambda expression: `(parameter, repository, userInfo) => { C# code }`.
  The properties on the action are available in the lambda expression as the first parameter.
  The action is available in Web API.
* **UseExecutionContext**  `<Action>` - Adds an additional input parameter "ExecutionContext" to the lambda expression.
  This concept is obsolete, repository member `_executionContext` should be used instead.
* **BeforeAction**  `<Action>.<name> <code snippet>` - It allows you to add a code snippet that will be executed before the Action.

## Claims and permissions

* **CustomClaim** `<custom resource name> <custom action or claim right>` - Add an additional security claim.
  It can be assigned by administrator to a user or a role, and verified by IAuthorizationManager.
  See [Basic permissions](Basic-permissions) for more info.
  Note: in the DSL script it must be placed outside of a Module.
* [**RowPermissions**](RowPermissions-concept) - Restricts access for selected users to a subset of the entity's records.
  * **AutoInheritRowPermissions** -  Each detail data structure in the module will inherit row permissions from it's mater data structure. Each extension in the module will inherit row permissions from it's base data structure.
  * **AutoInheritRowPermissionsInternally** -  similar to `AutoInheritRowPermissions` but it does not inherit row permissions from other modules.
  * **Allow** - Allows read and write access to selected records
  * **AllowRead** - Allows read access to selected records
  * **AllowWrite** - Allows write access to selected records
  * **Deny** - Denies read and write access to selected records
  * **DenyRead** - Denies read access to selected records
  * **DenyWrite** - Denies write access to selected records
  * Setting row permissions inheritance explicitly for selected entities
    * **InheritFrom**
    * **InheritFromBase** - can be used on a Browse data structures and on entities with Extends concept
  * **SamePropertyValue** - Used for internal optimizations when a property on one data structure returns the same value

See more on [User authentication and authorization](User-authentication-and-authorization).

## Low-level object model concepts

See the full article: [Low-level object model concepts](Low-level-object-model-concepts).

One of the basic principles in Rhetos framework is to allow developers to work in a classic way, coding the features directly in C# or SQL, by using the low-level concepts.

* **SaveMethod** - Allows developers to extend entity's Save method, by injecting a custom C# code that will be executed when saving the records (inserting, updating and deleting). There are different extension points at different positions in the Save method, that are intended for inserting a code with different purpose
  * **ArgumentValidation** - To verify if the parameters could break the rest of the Save method's business logic. Use `OnSaveValidate` instead for standard data validations.
  * **Initialization** - To initialize or change the data, before saving it to the database. If possible, use `DefaultValue` instead.
  * **OldDataLoaded** - To initialize or change the data, before saving it to the database, if previous data state needs to be considered. See related LoadOldItems concept.
  * **OnSaveUpdate** - To modify data in other dependent entities that needs to be updated (recomputed) after the current Save operation. If possible, use `ComputedFrom` instead.
  * **OnSaveValidate** - To implement a custom data validation. If possible, use `InvalidData` instead, for standard data validations, or `RowPermissions` for user permissions.
  * **AfterSave** - The inserted code will be executed after validations.
  * **LoadOldItems** - Simple helper for reading an old version of the data that can be reused in different business rules. It will load the old data between `Initialization` and `OldDataLoaded`.
* **RepositoryUses** - Adds a member property of a given type to the repository class, and initializes it automatically from the dependency injection container.
* **RepositoryMember** -  Allows you to add an arbitrary code to the repository class body. This can simplify code reuse between multiple filters, actions and other features.

## Database objects

See the full article: [Low-level database development](Database-objects).

These concepts are used as a workaround for features that cannot be implemented with the available high-level concepts.

* **SqlProcedure** `<Module>.<name> <arguments> <body>`
* **SqlTrigger** `<Entity>.<name> <events> <body>`
* **SqlFunction** `<Module>.<name> <arguments> <body>`
* **SqlView** `<Module>.<name> <select statement>`
* **SqlIndex** `<Property>`
* **SqlIndexMultiple** `<Entity>.<list of properties>`
* **Clustered** `<SqlIndexMultiple>` - Marks the index as clustered.
* **SqlDefault** `<Property> <sql statement>` - Generates default constraint on the database column.
  This concept is used only for internal features implemented in SQL procedures and triggers.
  Note: It cannot be used for default field value when writing data to Web API or in object model,
  because the saved record will always have the property value set to NULL by ORM, even if the value is not provided.
* [**SqlObject**](SqlObject-concept) `<Module>.<name> <create sql statement> <remove sql statement>` - Create a custom database object
  that is not supported by other SQL concepts, for example a full-text search catalog.

Dependencies between the database objects are required to make sure that they will be created in the correct order.
Use the following concepts to define the dependencies:

* **SqlDependsOn** `<dependent object> <depends on Property>` - The object should be created in database *after* the given column is created.
* **SqlDependsOn** `<dependent object> <depends on DataStructure>` - The object should be created in database *after* the given table or view is created.
* **SqlDependsOn** `<dependent object> <depends on Module>` - The object should be created in database *after* all database objects in the given module are created.
* **SqlDependsOnID** `<dependent object> <depends on ID>` - Dependency on ID property. Use this instead of SqlDependsOn entity to avoid having dependencies to all properties of the entity.
* **SqlDependsOnIndex** `<dependent object> <depends on Index>`
* **SqlDependsOnView** `<dependent object> <depends on SqlView>`
* **SqlDependsOnFunction** `<dependent object> <depends on SqlFunction>`
* **SqlDependsOnSqlObject** `<dependent object> <depends on SqlObject>`
* **AutodetectSqlDependencies** `<database object>` - Automatically detects and generates dependencies (SqlDependsOn) for the given database object, by analyzing its SQL script.
* **AutodetectSqlDependencies** `<Module>` - Automatically detects and generates dependencies (SqlDependsOn) for all database objects in the module.

BeforeQuery - New low-level concepts:BeforeQuery with parameter and BeforeAction, for injecting code in business object model.

ImplementsQueryable - Spominje se u Migrating-Rhetos-applications-from-NHibernate-to-Entity-Framework

PessimisticLockingParent

RegisteredImplementation - 1 Registers the data structure (and it's repository) as the main implementation of the given interface.This allows for type-safe code in external business layer class library to have simple access to the generated data structure's class and the repository using predefined interfaces.
2 Used for exposing repositories of an entity that implements a given interface. It helps to keep algorithm implementations out of DSL scripts by providing statically-typed querying and saving of generated object model entities without referencing the generated assembly.

RelatedItem

Write

From - vezan za Browse ?

useExecutionContext

RowPermissionsRead

RowPermissionsWrite
