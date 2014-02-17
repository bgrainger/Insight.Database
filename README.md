# Insight.Database #

**Insight.Database** is a fast, lightweight, (and dare we say awesome) micro-orm for .NET. It's available as a [NuGet Package](http://www.nuget.org/packages/Insight.Database/).

Whoops. Forgot to mention easy. It's easy too. And you can also take control of pieces if you want to.

Let's say you have a database and a class and you want them to work together. Something like this:

	CREATE TABLE Beer ([ID] [int], [Type] varchar(128), [Description] varchar(128)) GO
	CREATE PROC InsertBeer @type varchar(128), @description varchar(128) AS
		INSERT INTO Beer (Type, Description) OUTPUT inserted.ID
			VALUES (@type, @description)
			GO
	CREATE PROC GetBeerByType @type [varchar] AS SELECT * FROM Beer WHERE Type = @type GO

	public class Beer
	{
		public int ID { get; private set; }
		public string Type { get; set; }
		public string Description { get; set; }
	}

	var beer = new Beer() { Type = "ipa", Description = "Sly Fox 113" };

Let's get Insight.Database:

	PM> Install-Package Insight.Database

Now, wire up those stored procedures to an interface with a single `connection.As<T>`:

	public interface IBeerRepository
	{
		void InsertBeer(Beer beer);
		IList<Beer> GetBeerByType(string type);
		void UpdateBeerList(IList<Beer> beerList);
	}

	var repo = connection.As<IBeerRepository>();

	repo.InsertBeer(beer);
	IList<Beer> beerList = repo.GetBeerByType("ipa");
	repo.UpdateBeerList(beerList);

Look, ma! No mapping code! (Plus, you can now inject that interface with a DI framework or mock the interface for testing.)

Want to work at a lower level? Let's call a stored proc with an anonymous object. (It also automatically opens and closes the connection.)

	// auto open/close
	conn.Execute("AddBeer", new { Name = "IPA", Flavor = "Bitter"});

Objects are mapped automatically. No config files, no attributes. Just pure, clean magic:

	// auto object mapping
	Beer beer = new Beer();
	conn.Execute("InsertBeer", beer);
	List<Beer> beers = conn.Query<Beer>("FindBeer", new { Name = "IPA" });

Yes, even nested objects. Insight will just figure it out for you:

	// auto object graphs
	var servings = conn.Query<Serving, Beer, Glass>("GetServings");
	foreach (var serving in servings)
		Console.WriteLine("{0} {1}", serving.Beer.Name, serving.Glass.Ounces);

Feel free to return multiple result sets. We can handle them:

	// multiple result sets
	var results = conn.QueryResults<Beer, Chip>("GetBeerAndChips", new { Pub = "Fergie's" }));
	IList<Beer> beer = results.Set1;
	IList<Chip> chips = results.Set2;

Or perhaps you want to return multiple independent recordsets, with one-to-many and one-to-one relationships. It's not possible! Or is it?

	var results = conn.Query("GetLotsOfStuff", parameters,
		Query.Returns(OneToOne<Beer, Glass>.Records)
			.ThenChildren(Some<Napkin>.Records)
			.Then(Some<Wine>.Records));
	var beer = results.Set1;
	var glass = beer.Glass;
	var napkins = beer.Napkins.ToList();
	var wine = results.Set2;

But...but how? To that, I say: it's magic! Insight can decode the records and figure out one-to-one and parent-child relationships automatically. Almost all the time, it does this with no work or configuration on your part. But if you have a wacky model, there are plenty of non-scary ways to tell Insight how to work with your data. Read the [wiki](https://github.com/jonwagner/Insight.Database/wiki) for details.

Full async support. Just add `Async`:

	var task = c.QueryAsync<Beer>("FindBeer", new { Name = "IPA" });

Send whole lists of objects to databases that support table parameters. No more multiple queries or weird parameter mapping:

	// auto table parameters
	CREATE TYPE BeerTable (Name [nvarchar](256), Flavor [nvarchar](256))
	CREATE PROCEDURE InsertBeer (@BeerList [BeerTable])
	List<Beer> beerList = new List<Beer>();
	conn.Execute("InsertBeer", new { BeerList = beerList });

Insight can also stream objects over your database's BulkCopy protocol. 

	// auto bulk-copy objects
	IEnumerable<Beer> beerList; // from somewhere
	conn.BulkCopy("Beer", beerList);

Oh, wait. You want to inline your SQL. We do that too. Just add Sql.

	var beerList = conn.QuerySql<Beer>("SELECT * FROM Beer WHERE Name LIKE @Name", new { Name = "%ipa%" });
	conn.ExecuteSql ("INSERT INTO Beer VALUES (ID, Name)", beer);

Seriously, everything just works automatically. But if you *really* want to control every aspect of mapping, see the [wiki](https://github.com/jonwagner/Insight.Database/wiki).

## Motto ##

Insight.Database is the .NET micro-ORM that nobody knows about because it's so easy, automatic, and fast, (and well-documented) that nobody asks questions about it on StackOverflow.

## Licensing ##

Insight.Database is available under any of the following licenses:

* The "Tell Everyone You Know How Awesome Insight Is" License
* The "Answer Every Stackoverflow ORM question with 'Use Insight.Database'" License
* The "Buy A Friend A Beer" License
* The "Do Some Good" (Karmic) License
* MS-Public License (a.k.a the "yes it's free, but don't try to steal the IP and charge for it or the universe will haunt you" License)

## Major Insight Releases ##

* v4.0 - Read one-to-one, one-to-many, and many-to-many relationships automatically, with ways to extend it.
* v3.0 - Support for most common database and tools. See [the list of supported providers](https://github.com/jonwagner/Insight.Database/wiki/Insight-and-Data-Providers).
* v2.1 - Automatically [implement an interface](https://github.com/jonwagner/Insight.Database/wiki/Auto-Interface-Implementation) with database calls.
* v2.0 - Performance and asynchronous release.
* v1.0 - Let the magic begin!

## Documentation ##

**Full documentation is available on the [wiki](https://github.com/jonwagner/Insight.Database/wiki)**

