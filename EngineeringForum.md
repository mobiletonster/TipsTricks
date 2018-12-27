## .Net Engineering Forum 2018 December 20
### C# Tips and Tricks

<iframe width="560" height="315" src="https://www.youtube.com/embed/39gYjr1uo-0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


## Transcript

C# is an elegant and powerful language. Today we are going to look at some cool tips and tricks in C# that you may want to use in your code.

Let’s begin with a simple Person class.

    public class Person{
        public Person() { }

        public string Name;
    }

### C# Fields

Name is a field, not a property. Let's create a new instance of `Person` and examine the `Name` field.

    var person = new Person();
    person.Name = "Wonder Man";

### C# Properties

This works as you would expect. Let's add a property.

    public class Person{
        public Person() { }

        public string Name;
        public string FirstName {get; set;}
        public string LastName {get; set;}
    }

A Property is actually an accessor method. It works the same as a field in this instance.

    person.FirstName = "Bob";
    person.LastName = "Woodward";

So why use a property? Properties give us more control.

    public class Person{
        public Person() { }

        public string Name;
        public string FirstName {get; set;}
        public string LastName {get; set;}
        public string FullName {get {return FirstName + " " + LastName;}}
    }

Notice `FullName` lacks a setter. 

    person.FullName = "Mister Bob Woodward";

The IDE tells us this is a problem because it is a read only property. Let's just output the `FullName` to the Console and see what we get:

    Console.WriteLine(person.FullName);

Let's add `Gender` with a private setter.

    public string Gender {get; private set;}

It behaves similar to `FullName`, but gives a different error.

    person.Gender = "Male";

How do we set this value? We can set it in the Constructor.

    public Person(string gender){
        Gender = gender;
    }

Now we need to create a new instance and use the constructor overload with `gender` as a parameter;

    var person = new Person("Male");

    Console.WriteLine(person.Gender);

### C# Enumerations
Using a type of string may be too loose for `Gender`. Let's use an enumeration instead. We'll create a new GenderType.cs file.
    
    public enum GenderType
    {
        Male,
        Female
    }

    public GenderType Gender {get; private set; }

    public Person(GenderType gender){
        Gender = gender;
    }

    Console.WriteLine(person.Gender);

Let's use explicit values for `GenderType` enumeration. This allows for expansion of future gender types.

    public enum GenderType
    {
        Male=10,
        Female=20
    }

    Console.WriteLine(person.Gender + " " + (int)person.Gender);

Notice when we output the value to the console, we get the friendly enumeration name rather than value unless we cast the value as an integer.

### C# Full Properties with Backing Fields

Let's use a backing field for a property called `Title`.

    private string _title;

    public string Title {
        get {return _title;}
        set {_title=value;}
    }

Using the property feels the same:

    person.Title = "Mister";

However, this structure allows us to inspect the incoming (or outgoing) values and modify them if need be. Let's assume that we always want to change 'Mister' to 'Mr.'. We can do this. Let's create a private method to handle this.

    private string SanitizeTitle(string incoming){
            return incoming.Replace("Mister", "Mr.", StringComparison.InvariantCultureIgnoreCase);
    }

    set { _title=SanitizeTitle(value)}

And we will try it.

    person.Title="Mister";

    Console.WriteLine(person.Title);

### C# New Lambda Syntax in Properties

Cool. We should add title to the `FullName` field. We can write it better using a 'fat arrow' syntax for readonly properties.

    public string FullName => Title + " " + FirstName + " " + LastName;

### C# Default Values for Properties

Let's create properties with default values.

    public class Person{
        private string _title;
        public Person() { }
        public Person(string gender){
            Gender=gender;
        }
        public string Name;
        public string FirstName {get; set;}
        public string LastName {get; set;}
        public string FullName => Title + " " + FirstName + " " + LastName;
        public string Title {
            get {return _title;}
            set {_title=SanitizeTitle(value);}
        }
        public DateTime ModifiedDate {get;set;} = DateTime.UtcNow;
        public DateTime CreatedDate {get;set;} = DateTime.UtcNow;
    }

If we don't provide a value, these fields will automatically get timestamped with the current UTC DateTime.

Let's write to the console to see the values.

    person.ModifiedDate = DateTime.UtcNow.AddDays(3);

    Console.WriteLine(person.CreatedDate + " " + person.ModifiedDate);

### C# Overrides

Wouldn't it be nice if we could just use ToString() to get values out? Good news, ToString() is build into every object.

    Console.WriteLine(person.ToString());

Bad news, the default implementation for classes is to output the class name. Not real useful.

We can replace the default implementation.

    public string ToString(){
        return Title + " " + FirstName + " " + LastName + " " + Gender;
    }

You may notice that the IDE is warning us that we are hiding an already implemented method in our base object (which is object). It will still work, but we can fix this by using the `override` keyword.

    public override string ToString() {
        return Title + " " + FirstName + " " + LastName + " " + Gender;
    }

Now call `ToString()` and see what happens.

    Console.WriteLine(person.ToString());

### C# Expression Body Definition / String Interpolation

Much better. To make it easier to read, maybe we should separate by commas instead. Lets create a `ToCommaString()` method instead. Lets use an Expression Body definition instead and use string interpolation because we are cool.

    public string ToCommaString() => $"{Title}, {FirstName}, {LastName}, {Gender}";

And run it:

    Console.WriteLine(person.ToCommaString());

We are definitely cool. But, it is a pain to keep adding these fields over and over again. Let's use reflection to do this for us.

    public string ToCommaDelimited()
    {
        string output = string.Empty;
        foreach(var pinfo in this.GetType().GetProperties())
        {
            output += pinfo.Name + ": " + pinfo.GetValue(this) + ", ";
        }
        return output;
    }   

    Console.WriteLine(person.ToCommaDelimited());

### C# Obsolete Keyword

When we run this, we get more key value pairs...nice. Let's let people know that the `ToCommaString()` isn't as cool and will probably be removed in the future.

    [Obsolete("It would be cooler for you to use ToCommaDelimited(). Just sayin.")]

Notice that the IDE gives a squiggly underline warning that the `ToCommaString()` is `obsolete`.

If we want to the compiler to fail, we can use the optional boolean parameter.

    [Obsolete("It would be cooler for you to use ToCommaDelimited(). Just sayin.",true)]

Wow! We are super close to having a Json outputter. Let's add one.

    public string ToJson()
    {
        string output = "{";
        foreach (var pinfo in this.GetType().GetProperties())
        {
            output += $"'{pinfo.Name}':'{pinfo.GetValue(this)}',";
        }
        output = output.Substring(0, output.Length - 1) + "}";
        return output;
    }

Ok, so Json is great for computers, but not great for humans. We'd rather have Line Feeds to break things up. Let's add one.

    public string ToLF()
    {
        string output = string.Empty;
        foreach (var pinfo in this.GetType().GetProperties())
        {
            output += $"{pinfo.Name}: {pinfo.GetValue(this)}\n";
        }
        return output;
    }

### C# Ternary or Conditional Operator

Ok. Let's add a `BirthDate` field and an `Age` calculation field. Can we use a Fat Arrow syntax for our property? Sure, let's also use a Ternary or Conditional Operator `?:`

    public DateTime BirthDate { get; set; }
    public int Age => (DateTime.UtcNow.DayOfYear >= BirthDate.DayOfYear)?(DateTime.UtcNow.Year - BirthDate.Year):(DateTime.UtcNow.Year-BirthDate.Year-1);

Age is a function of BirthDate and the current date...you age on or after your birthday of the year. If we choose a birthday ealier than today, versus after today, the age in years will be a difference of 1.

Ok.  It isn't perfect. There are some scenarios that don't make sense, but for now, lets not worry about it.

### C# More Enums

Let's add a Social Media Type.

    public enum SocialMediaType
    {
        Facebook,
        Twitter,
        Instagram,
        GooglePlus,
        LinkedIn,
        Pinterest,
        Reddit,
        YouTube,
        Tumblr
    }

    public SocialMediaType SocialMedia {get;set;}

    person.SocialMedia = SocialMediaType.Twitter;

What happens if we don't assign a value? hmm... Do we want everyone to be Facebook by default? Probably not.

### C# Nullable Value Operator (?)

We can make this type optional, or nullable.

    public SocialMediaType? SocialMedia {get; set;}

Now if we don't assign a value, we don't get a value. Notice also, that the values aren't necessarily mutually exclusive? We could break this into separate, mutually exclusive enums...

### C# Enum Flags and Bitwise Operations

...but we want to demonstrate `Flags`. Flags need to be explicitly set to binary positional values like this:

    [Flags]
    public enum SocialMediaType
    {
        Facebook=1,
        Twitter=2,
        Instagram=4,
        GooglePlus=8,
        LinkedIn=16,
        Pinterest=32,
        Reddit=64,
        YouTube=128,
        Tumblr=256
    }

With flags set, we can combine and add more values using the `Bitwise OR` operator.

    person.SocialMedia = SocialMediaType.Twitter | SocialMediaType.Pinterest; 

also:

    person.SocialMedia = person.SocialMedia | SocialMediaType.Facebook;

    person.SocialMedia |= SocialMediaType.Instagram;


Now when we run this, it shows that the person has more than one flag set to on.

We can perform checks like this:

    var hasInstagram = person.SocialMedia?.HasFlag(SocialMediaType.Instagram);

### C# Null Conditional Operator (Elvis Operator)

Notice the use of the `Null Conditional Operator` `?.`.

    if(hasInstagram.HasValue && hasInstagram.Value){
        Console.WriteLine("Person has Instagram!");
    }

    if(hasInstagram.GetValueOrDefault()){
        Console.WriteLine("Person has Instagram!");
    }

### C# Null Coalescing Operator

or use the null coalescing operator `??`.

    if(hasInstagram??false){
        Console.WriteLine("Person has Instagram!");
    }

better yet, move that up a line:

    var hasInstagram = person.SocialMedia?.HasFlag(SocialMediaType.Instagram)??false;

    if(hasInstagram){
        Console.WriteLine("Person has Instagram!");
    }

### Console Color Options

spice it up with some color:

    if (hasInstagram)
    {
        Console.ForegroundColor = ConsoleColor.Green;
        Console.WriteLine("Person has Instagram!");
        Console.ForegroundColor = ConsoleColor.White;
        Console.BackgroundColor = ConsoleColor.DarkBlue;
    }

We can control the colors of the Console! Nice. Makes it easier to find what we are looking for. We can also reset it to default in one operation:

    Console.ResetColor();

### C# Bit Shift Operator with Flags    

Setting flags to binary increments could get unwieldy and hard to remember. We can leverage a Bit Shift operator to make it easier .

    [Flags]
    public enum SocialMediaType
    {
        Facebook = 1 << 0,
        Twitter = 1 << 1,
        Instagram = 1 << 2,
        GooglePlus = 1 << 3,
        LinkedIn = 1 << 4,
        Pinterest = 1 << 5,
        Reddit = 1 << 6,
        YouTube = 1 << 7,
        Tumblr = 1 << 8,
        //combinations can be in the enum too
        MostPopular = Facebook | Twitter | Instagram
    }

We can use math to subtract a value:

    person.SocialMedia -= (int)SocialMediaType.Twitter;

Or using bitwise operations, we can use a combination of & (bitwise and) and ~ (bitwise not);

    person.SocialMedia &= ~SocialMediaType.Facebook;

Before we do the next block, let's move all our code into a static method called TestPerson().

    static void TestPerson() {
        ...
    }

### C# Show Full Stack Trace With Exceptions

Ok. Let's look at exceptions. We want to cause an exception and see what happens to the stack trace.

    person.Title=null;

If we wrap this in a try-catch block, let's see what happens.

    try {
        person.Title=null;
    } catch(Exception ex){
        throw ex;
    }

and in the main method, wrap the call to TestPerson() in a try-catch block.

    try {
        TestPerson();
    } catch(Exception ex){
        Console.WriteLine(ex.StackTrace);
    }

Now watch what happens when we remove the throw ex; line from our first try-catch block.

    try {
        person.Title=null;
    } catch(Exception ex)
    {
        throw;
    }
    
We retain a complete stack trace!

### C# IEnumerable<T> magic

Let's generate list of people. We will use the object initializer to initialize make it cleaner:

    static List<Person> GeneratePeople()
    {
        var persons = new List<Person>();
        persons.Add(new Person(GenderType.Male) { FirstName="Tony", LastName="Spencer", Title="Mister",BirthDate=DateTime.Parse("Jan 1, 1970"), SocialMedia=SocialMediaType.MostPopular });
        persons.Add(new Person(GenderType.Female) { FirstName = "Amber", LastName="Heard", Title="Miss", BirthDate=DateTime.Parse("Apr 22, 1986"), SocialMedia=SocialMediaType.MostPopular });
        persons.Add(new Person(GenderType.Male) { FirstName = "Matt", LastName = "Damon", Title = "Mister", BirthDate = DateTime.Parse("Oct 8, 1970"), SocialMedia = SocialMediaType.Twitter });
        persons.Add(new Person(GenderType.Female) { FirstName="Eva", LastName="Green",Title="Ms.",BirthDate=DateTime.Parse("Jul 6, 1980"), SocialMedia=SocialMediaType.Facebook|SocialMediaType.Instagram });
        persons.Add(new Person(GenderType.Male) { FirstName = "Chris", LastName = "Pine", Title = "Mr.", BirthDate = DateTime.Parse("Aug 26, 1980"), SocialMedia = SocialMediaType.MostPopular });
        persons.Add(new Person(GenderType.Female) { FirstName = "Amy", LastName = "Farrah Fowler", Title = "Dr.", BirthDate = DateTime.Parse("Dec 12, 1975"), SocialMedia = SocialMediaType.LinkedIn });
        persons.Add(new Person(GenderType.Male) { FirstName = "Elton", LastName = "John", Title = "Sir", BirthDate = DateTime.Parse("Mar 25, 1947"), SocialMedia = SocialMediaType.MostPopular });
        persons.Add(new Person(GenderType.Female) { FirstName = "Phil", LastName = "McGraw", Title = "Dr.", BirthDate = DateTime.Parse("Sep 1, 1950"), SocialMedia = SocialMediaType.YouTube });

        return persons;
    }

### C# List<T>.ForEach LINQ Method

Now let's create a test method we can call `TestPeople()`.

    static void TestPeople(){
        List<Person> people = GeneratePeople();

        people.ForEach(p=> Console.WriteLine(p.ToCommaDelimited()));
    }

When we run this, it is a bit hard to read because we get all the field names. Let's make an option to turn that part off.

    public string ToCommaDelimited(bool includePropertyName=true)
    {
             string output = string.Empty;
            foreach(var pinfo in this.GetType().GetProperties())
            {
                var prefix = includePropertyName ? (pinfo.Name + ":") : "";
                output += prefix + pinfo.GetValue(this) + ", ";
            }
            return output;       
    }

    people.ForEach(p => Console.WriteLine(p.ToCommaDelimited(includePropertyName:false)));

### C# Named Parameter Option

Notice we used the named parameter option rather than simply passing an arbitrary `false` value.

Next, we want to create a method to extract only people that are older than 35 and have a Twitter account.

    public static List<Person> GetOldTwitterPeople(List<Person> persons)
    {
        List<Person> temp = new List<Person>();
        foreach(var p in persons)
        {
            var hasTwitter = p.SocialMedia?.HasFlag(SocialMediaType.Twitter) ?? false;
            if(p.Age>35 && hasTwitter)
            {
                temp.Add(p);
            }
        }

        return temp;
    }

This is a common pattern that we see. Create a temp list to put the filtered/matching set of persons. Then return that temp list. 

### C# IEnumerable<T> Yield Keyword

While this works, you could refactor this slightly to avoid creating a temporary variable leveraging the `yield` keyword.

    public static IEnumerable<Person> GetOldTwitterPeople(IEnumerable<Person> persons)
    {
        foreach(var p in persons)
        {
            var hasTwitter = p.SocialMedia?.HasFlag(SocialMediaType.Twitter) ?? false;
            if (p.Age > 35 && hasTwitter)
            {
                yield return p;
            }
        }
    }

Notice we change the return type to IEnumerable<Person> which allows us to use the `yield return p` statement.

## Conclusion

C# is an amazing, powerful language. We looked at a few tips and tricks that can help improve our code. We have barely scratched the surface. We hope to have more installments of tips and tricks in the future. In the mean time, happy coding!


