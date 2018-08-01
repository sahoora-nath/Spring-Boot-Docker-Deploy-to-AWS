
### Binary Search on Arrays and Collections
The `Collections` and the `Arrays` class both provide methods that allow you to search for a specific element. While searching the following rule apply.

- Search is performed using `binarySearch()` method.
- Successful search return the `int` index of the element being searched.
- Unsuccessful search returns -1
- The collection/array being searched must be sorted before you can search it.
- If you attempt to search an array/collection that has not already been sorted, the results of the search will not be predicted.
- If the collection/array was sorted using natural order, it must be searched in natural order (can be done by NOT passing `comparator` as an argument to `binarySearch()`).
- If the collection/array was sorted using a `comparator`, it must be searched using the same `comparator`

:tada: `comparator` can't be used when searching arrays of primitives.

```Java
import java.util.Arrays;

public class SearchObjArr {
    public static void main(String[] args) {
        String[] sa = {"one", "two", "three", "four"};

        Arrays.sort(sa);
        for (String s : sa) {
            System.out.print(s + " ");
        }

        System.out.println("\none = "+ Arrays.binarySearch(sa, "one"));

        //Now reverse the sort using comparator.
        Arrays.sort(sa, (a,b) -> b.compareTo(a));
        for (String s : sa) {
            System.out.print(s + " ");
        }
        //Attempt to search array which is sorted using a comparator and didn't pass the comparator to binarySearch().
        System.out.println("\none = "+ Arrays.binarySearch(sa, "one"));
        //hence incorrect result.

        //Now we pass the comparator to binary search
        System.out.println("\ncomparator passed to binary search");
        System.out.println("one = "+ Arrays.binarySearch(sa, "one", (a,b) -> b.compareTo(a)));
    }
}
```
This produces the following output:
```Console
four one three two
one = 1
two three one four
one = -1

comparator passed to binary search
one = 2
```
