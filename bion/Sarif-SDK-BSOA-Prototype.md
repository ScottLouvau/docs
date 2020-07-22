We've created a prototype version of the Sarif SDK which uses BSOA underneath the object model. This version can be used instead of the normal SDK, but should be tested to ensure correct behavior in your specific scenario. It shouldn't be used in production environments yet.

## Advantages
The BSOA object model uses significantly less memory (60% - 90%, usually) and can be loaded from the SARIF JSON format about twice as fast as the old object model.

SarifLogs can be written in a new BSOA binary form, which is about 33% smaller than the unindented JSON SARIF and can be loaded and saved 100x faster (800 MB - 1,900 MB/s). SarifLog.Save() and SarifLog.Load() use this format if the file extension is ".bsoa". The binary format may change, so store JSON copies of important data.


## Disadvantages
The BSOA object model does not yet have garbage collection. Sarif OM objects created won't be garbage collected unless every object under a SarifLog is unreferenced. When a log is written in the BSOA form, objects which were created but not put into collections will be included in the written file. This can be mitigated by calling SarifLog.DeepClone() just before writing, which will copy only included objects.

The BSOA object model is slower once loaded for some scenarios. Strings, in particular, must be converted from UTF-8 bytes to .NET strings each time they are accessed. Cache strings in variables to avoid reading them from the object model multiple times within the same method.


## Recommended Scenarios
The BSOA version of the SDK is excellent for read-only usage of logs (like querying them), simple changes (like Multitool rewrite), and working with large logs more quickly. The binary format is very useful for logs which need to be loaded multiple times (like for multiple Multitool steps in succession) or when your code will be the only reader and writer of a set of logs.

## Usage Caveats
See [BSOA Caveats](https://github.com/microsoft/bion/wiki/BSOA-Caveats) for details.

#### Use constructors which take a SarifLog when possible.
BSOA object data is stored in 'tables' under a specific root SarifLog. Use the constructor which takes a SarifLog to ensure the objects are created under the correct root so that the data doesn't have to be copied later.

#### Fully initialize objects and fully populate collections before setting them.
BSOA copies data from external types (List, Dictionary) to an internal representation on set. Changes made to the collection after setting it won't take effect on the copy in your SarifLog model. BSOA also copies objects from a different SarifLog root on set, so instances can't be reused across SarifLogs and changes made to an object after setting it won't effect both SarifLog copies.

#### Cache strings you retrieve.
BSOA stores strings as UTF-8 and must convert them to .NET strings when retrieved. Keep returned strings in a variable rather than getting them repeatedly to avoid extra conversions.

#### Avoid creating objects you won't use.
BSOA objects take memory on the SarifLog they belong to until the whole log is garbage collected. If you are creating many Results but will only add some to the output log, create a temporary SarifLog to hold the Results and periodically clear it with SarifLog.DB.Clear() (or by re-creating the temporary log and letting the old one be collected) to avoid leaking memory.

#### Dictionaries are slow.
The BSOA Dictionary implementation doesn't organize keys by hash code (yet), so it must walk each key/value pair to look up a key. Keep Dictionaries small and avoid accessing keys repeatedly on them.

### Other Differences
* Sarif objects now directly support Equals() and GetHashCode(), which compare the values of all properties under an object.
* The object model will not skip serializing non-empty collections which have all default objects; keep the collections empty or null to avoid serializing them.
* The binary format serializes enums using the numeric values; ensure these are stable to avoid compatibility problems.