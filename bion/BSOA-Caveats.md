BSOA object models look very similar to normal object models, but behave differently in a few keys ways.

# Dos
- Use the constructors which take a root object.
- Fully initialize all properties before adding the object to a collection or setting a reference to it.
- Convert DateTimes to UTC before setting.
- Cache string values retrieved from the object model in local variables if they are used repeatedly.
- Keep Dictionaries small and avoid accessing keys repeatedly.

# Details
### Construction
All BSOA objects must be tied to a Table which holds the data. Constructors which take the root object will create it in the correct table. Other constructors will guess, putting the object under the last created root for the current thread.

When objects from one root are set on the object model of another, they are copied. This takes time, and means that changes to the object after it is set will not apply to both copies.

### Copy-on-Set
BSOA object models copy values into internal data structures on set. This means that if you make changes to a value after setting it, the changes will not take effect on the copy in BSOA. For example, if you set a BSOA object property to a List<string> and then add more strings to it, the strings added after you set the list will not be on the BSOA copy.

### DateTimes are UTC
All DateTimes in BSOA are stored as UTC. If you set a DateTime BSOA property and read it back, the value returned will have been converted to a Universal time.

### String Conversion
Strings inside BSOA are stored as UTF-8 bytes, and must be converted to .NET strings when they are retrieved. Keep a copy of the string in a variable to avoid repeated string conversions when you're referencing the string multiple times.

#### Dictionaries are slow
The BSOA Dictionary implementation doesn't organize keys by hash code (yet), so it must walk each key/value pair to look up a key. Keep Dictionaries small and avoid accessing keys repeatedly on them.

### Object Lifetime
All BSOA objects which are created will end up in the Table. If you create an object instance but never put it into a collection reachable from the root object, BSOA must garbage collect it. If you are creating many objects but will only keep some of them, it's best to create a "scratch" root object. When you add instances to collections in your "real" object, BSOA will copy the data to the tables to be output.

In addition, BSOA objects are not garbage collected normally during execution. Any instances kept in memory will hold the root object and all other objects created in the table. If you want to create objects as you go without keeping them all in memory, you need to avoid keeping instances. If you're serializing them periodically, you should call root.DB.Clear() after each serialization to get rid of previously added data.