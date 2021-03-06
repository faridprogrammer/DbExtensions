[DbExtensions](http://maxtoroq.github.io/DbExtensions/) — The SQL framework for .NET
====================================================================================
DbExtensions is a data-access API with a strong focus on **query composition, granularity and code aesthetics**.
It supports both POCO and dynamic (untyped) mapping.

Get it now! using NuGet
-----------------------
```powershell
Install-Package DbExtensions
```

Querying with SqlSet
--------------------
```csharp
var db = new Database("name=Northwind");

SqlSet<Product> products = db.From<Product>("Products");
SqlSet<Product> productsToReorder = products.Where("UnitsInStock < {0}", 10);

if (productsToReorder.Any()) {

   SqlSet<Product> top5WithLowestStock = productsToReorder.OrderBy("UnitsInStock").Take(5);
   Product first = top5WithLowestStock.First();

   if (top5WithLowestStock.Count() > 1) {
      Product second = top5WithLowestStock.Skip(1).First();
   }
}
```
[SqlSet](docs/SqlSet.md) provides a LINQish API for making queries, but using SQL instead of lambda expressions. The above code executes the following queries:

```sql
SELECT (CASE WHEN EXISTS (
   SELECT *
   FROM Products
   WHERE UnitsInStock < @p0) THEN 1 ELSE 0 END)
-- @p0: Input Int32 (Size = 0) [10]
-- [-1] records affected.
SELECT *
FROM (
   SELECT *
   FROM Products
   WHERE UnitsInStock < @p0
   ORDER BY UnitsInStock
   LIMIT @p1) dbex_set5
LIMIT @p2
-- @p0: Input Int32 (Size = 0) [10]
-- @p1: Input Int32 (Size = 0) [5]
-- @p2: Input Int32 (Size = 0) [1]
-- [-1] records affected.
SELECT COUNT(*)
FROM (
   SELECT *
   FROM Products
   WHERE UnitsInStock < @p0
   ORDER BY UnitsInStock
   LIMIT @p1) dbex_count
-- @p0: Input Int32 (Size = 0) [10]
-- @p1: Input Int32 (Size = 0) [5]
-- [-1] records affected.
SELECT *
FROM (
   SELECT *
   FROM Products
   WHERE UnitsInStock < @p0
   ORDER BY UnitsInStock
   LIMIT @p1) dbex_set6
LIMIT @p2
OFFSET @p3
-- @p0: Input Int32 (Size = 0) [10]
-- @p1: Input Int32 (Size = 0) [5]
-- @p2: Input Int32 (Size = 0) [1]
-- @p3: Input Int32 (Size = 0) [1]
-- [-1] records affected.
```

Building queries with SqlBuilder
--------------------------------
```csharp
var query = SQL
   .SELECT("p.ProductID, p.ProductName, s.UnitPrice, p.CategoryID")
   ._("c.CategoryID AS Category$CategoryID, c.CategoryName AS Category$CategoryName")
   .FROM("Products p")
   .JOIN("Categories c ON p.CategoryID = c.CategoryID")
   .WHERE()
   ._If(categoryId.HasValue, "p.CategoryID = {0}", categoryId);

IEnumerable<Product> products = db.Map<Product>(query);
```
With [SqlBuilder](docs/SqlBuilder.md) you have complete control of the executing SQL.

Changing data
-------------
```csharp
public class NorthwindDatabase : Database {
   
   public SqlTable<Product> Products { // SqlTable inherits from SqlSet
      get { return Table<Product>(); } 
   }

   public NorthwindDatabase() 
      : base("name=Northwind") { }
}

var db = new NorthwindDatabase();

Product prod = db.Products.Find(1);
prod.UnitPrice = prod.UnitPrice * 1.1;

db.Products.Update(prod);
```
You can also use SqlBuilder to build insert, update and delete commands.

Features
--------
- Deferred execution
- POCO and dynamic mapping for queries
  - Mapping to properties (including complex)
  - Mapping to constructor arguments
- Attributes or XML mapping for inserts, updates and deletes, using [System.Data.Linq.Mapping](http://msdn.microsoft.com/library/system.data.linq.mapping) (LINQ to SQL mapping)
- Generic and non-generic APIs (for when the type of the entity is not known at build time)
- Automatic connection management (no need to explicitly open connection, but you are allowed to)
- Eager loading
- Optimistic concurrency (using version column)
- Batch commands
- Profiling
- Provider-independent (tested against SQL Server, SQL Server Compact, MySQL and SQLite)

Not included
------------------------
DbExtensions doesn't provide the following functionality:

- Unit of work
- Change tracking
- Identity map
- Lazy loading

Resources
---------
- [Documentation](docs/README.md)
- [Ask for help](https://github.com/maxtoroq/DbExtensions/issues?q=is%3Aissue+label%3Aquestion)
- [Report an issue](https://github.com/maxtoroq/DbExtensions/issues)

[![Donate](https://www.paypalobjects.com/en_US/i/btn/btn_donate_SM.gif)](http://maxtoroq.github.io/p/donate.html)
[![Flattr this](https://api.flattr.com/button/flattr-badge-large.png)](http://flattr.com/thing/1761218/DbExtensions)
