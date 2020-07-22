BSOA is a framework for creating object models which use the ["struct-of-arrays"](https://en.wikipedia.org/wiki/AoS_and_SoA) (SoA) design pattern. 

BSOA object models take up substantially less memory than normal object models and support binary serialization with gigabyte-per-second read and write speeds. 

In our original use case, SARIF, the object model typically takes 75% less memory, the binary file is 33% smaller than unindented JSON, and logs in the binary format can be read and written 100x faster than the JSON form.

### Normal vs. Struct-of-Array object models
Object model classes normally store data directly in fields, and may store primitive values (numbers, strings, booleans), references to other classes, or collections of these things. Hierarchies of these classes can be easily constructed in code, and libraries like Newtonsoft.Json provide easy serialization of them to and from the JSON file format.

```csharp
    public class Result
    {
        public string RuleId { get; set; }
        public Message Message { get; set; }
        public string Guid { get; set; }
    }
```

However, this design means that every logical object in the data model requires a separate real object in memory. Each object must be constructed, has object overhead, must be managed by the garbage collector, and takes space for all of the fields defined on it, whether or not they are used. For complex object models with many fields and deep layers of classes, the overhead can take more space than the useful data.

Serialization from JSON is straightforward, but relatively slow. The serializer must identify each property in the JSON, parse the single value, and then call the setter for the corresponding class field. Newtonsoft.Json can parse JSON at about 100 MB/s, but slows to 25 MB/s or less when filling out an object model, particularly one with many small values to parse and many data classes to construct.

In the "struct-of-arrays" model, each data model class has a 'Table' class. The Table class holds data for many instances of the data class. Instead of single fields, the Table class has an array for each column. A data model class 'instance' really becomes just a reference to the Table and a row index. Each field on the class becomes a property which retrieves the value from the row index in the Table class array for the field name.

```csharp
    public struct Result
    {
        internal readonly ResultTable _table;
        internal readonly int _index;

        public string RuleId
        {
            get => _table.RuleId[_index];
            set => _table.RuleId[_index] = value;
        }

        public Message Message { ... }
        public string Guid { ... }
    }

    internal class ResultTable
    {
        internal SarifLogDatabase DB { get; }

        public IColumn<string> RuleId { get; }
        public IColumn<int> Message { get; }
        public IColumn<string> Guid { get; }
    }

```

### Why would you use Struct-of-Arrays? 

First, in this model, unused fields use no memory. The column array for 'Guid' can be left null, taking up no space, and the empty column can return the default value when accessed. 

Second, the Result objects no longer need to be kept. The data is all on the Table class, so we only need to keep it in memory. There's only the one Table class and the column arrays, so there's a low, fixed amount of object overhead for any number of Result instances. 

Third, storing values together by column can allow us to compress values. For example, when storing a set of strings together, we can convert them from .NET's UTF-16 representation to UTF-8 in a shared byte array, and then convert individual values back to strings when accessed. Strings become drastically smaller, as they can use just one byte per character instead of two, and 24 bytes of overhead and an 8 byte string pointer are eliminated for every string. For columns with few distinct values, we can even store each distinct value only once, and have each row identify which value it contains.

Fourth, by storing the data by column, we have a fast option for serializing it. Rather than writing values individually, we can write out a whole column array in a single method call. The binary format 'BSOA' implemented by the library does just that, writing some metadata for Tables and Columns and then writing each column array as a block. BSOA is able to read and write at over 1 GB/s single-threaded as a result of this design.


### BSOA Core Library

Clearly, the struct-of-arrays design requires much more code. The BSOA library helps by providing column classes for base types (numbers, bool, string, DateTime, List, Dictionary), implementing binary serialization, and providing a generator to generate the object model for you. The generator takes a simple schema and configurable templates, allowing you to customize how the generated classes look. It's also straightforward to implement your own column types, if needed, and use them from the generated model. The generated classes are partial classes, which makes it easy to combine generated code with handwritten code for other operations on the classes. 