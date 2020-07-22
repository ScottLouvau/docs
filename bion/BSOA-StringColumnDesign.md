### Goals

- Handle any columns with variable length values (StringColumn, ByteArrayColumn, ReferenceColumn)
- Provide low overhead to track positions and lengths of values.
- Provide constant time access to positions and lengths when reading.
- Provide inexpensive writing.
- Handle values of common sizes inline (not stored individually).

 

### Design

A VariableLengthColumn is split up into **chapters** of 1,024 rows which are further split into **pages** of 32 rows. Values under length 2,048 are stored together in a "small value" array. Values that are length 2,048 or larger are stored in a second "large value" array.

Since small values are limited to 2,047 bytes, a page of small values can be at most 2,047 x 32 = 65,504 bytes, and a chapter can be at most 2,047 x 1,024 = 2,096,128 bytes.

For a column of UTF-8 strings, therefore, we need the following arrays:

- byte[] SmallValueArray;
- ushort[] ValueEndInPage; //     One per Row
- int[] PageStartInChapter; //     One per Page
- Dictionary<int, byte[]>     LargeValueDictionary;

 

In C# we can't keep one of each array for the whole column, because the SmallValueArray is limited to 2 GB, which could be hit after only one million rows.

Therefore, we keep separate arrays per chapter. We have the overhead of three arrays and the Dictionary for each 1,024 rows.

 

### Per Chapter

| Component                                     | Overhead                           |
| --------------------------------------------- | ---------------------------------- |
| ushort[]  ValueEndInPage                      | 8 + 24 + (2 x  1,024) = 2,080      |
| int[]  PageStartInChapter                     | 8 + 24 + (4 x  (1,024 / 32)) = 160 |
| byte[]  SmallValueArray                       | 8 + 24                             |
| Dictionary<int,  byte[]> LargeValueDictionary | 8 + 24                             |

 Stored this way, overhead per value is (2,080 + 160 + (8 + 24) x 2) = 2,304 / 1,024 = 2.25 bytes.




```csharp
Value[i] => {
  int chapterIndex = I / 1024;
  int indexWithinChapter = I & 1023;
   
   Chapter c = Chapters[chapterIndex];
 
   if(c.LargeValueDictionary?.TryGetValue(indexWithinChapter, out byte[] singleValue)
   {
​      return new String8(singleValue, 0, singleValue.Length);
   }
 
  int pagePosition = c.PageStartInChapter[indexWithinChapter / 32];
  int startInPage = (indexWithinChapter == 0 ? 0 : ValueEndInPage[indexWithinChapter - 1]);
  int length = ValueEndInPage[indexWithinChapter] - startInPage;
  return new String8(c.SmallValueArray, pagePosition + startInPage, length);
}
```


All changed values are stored in the LargeValueDictionary. A flag is set per Chapter indicating if there are changed values to be merged on write. 