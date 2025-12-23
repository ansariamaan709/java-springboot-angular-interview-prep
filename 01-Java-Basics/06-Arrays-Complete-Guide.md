# Arrays in Java - Complete Deep Dive

## Table of Contents

1. [Introduction to Arrays](#introduction-to-arrays)
2. [Array Declaration and Initialization](#array-declaration-and-initialization)
3. [Array Operations](#array-operations)
4. [Multi-Dimensional Arrays](#multi-dimensional-arrays)
5. [Arrays Utility Class](#arrays-utility-class)
6. [Array Copying](#array-copying)
7. [Common Array Algorithms](#common-array-algorithms)
8. [Arrays vs ArrayList](#arrays-vs-arraylist)
9. [Interview Questions](#interview-questions)

---

## Introduction to Arrays

### What is an Array?

An array is a **container object** that holds a **fixed number of values** of a **single type**. The length of an array is established when the array is created and cannot be changed.

### Key Characteristics

1. **Fixed Size**: Once created, size cannot change
2. **Homogeneous**: All elements must be same type
3. **Indexed**: Elements accessed by index (0-based)
4. **Contiguous Memory**: Elements stored sequentially
5. **Object**: Arrays are objects in Java (even for primitives)

### Array Memory Layout

```
Stack                         Heap
┌─────────────────┐          ┌─────────────────────────────────────┐
│ arr (reference) │─────────►│ Array Object                        │
│    [0x1000]     │          │ ┌─────────────────────────────────┐ │
└─────────────────┘          │ │ length = 5                      │ │
                             │ ├────┬────┬────┬────┬────┐        │ │
                             │ │ 10 │ 20 │ 30 │ 40 │ 50 │        │ │
                             │ ├────┴────┴────┴────┴────┘        │ │
                             │ │ [0]  [1]  [2]  [3]  [4]         │ │
                             │ └─────────────────────────────────┘ │
                             └─────────────────────────────────────┘
```

---

## Array Declaration and Initialization

### Declaration Syntax

```java
public class ArrayDeclaration {
    public static void main(String[] args) {
        // Declaration styles (both are valid, first is preferred)
        int[] numbers;       // Preferred - type is "array of int"
        int numbers2[];      // C-style - variable is an array

        // Multiple declarations
        int[] a, b, c;       // All are arrays
        int d[], e, f[];     // Only d and f are arrays, e is int

        // Array of objects
        String[] names;
        Object[] objects;
        Integer[] integers;
    }
}
```

### Initialization

```java
public class ArrayInitialization {
    public static void main(String[] args) {
        // 1. Declaration + Creation (separate)
        int[] arr1;
        arr1 = new int[5];  // Default values: 0, 0, 0, 0, 0

        // 2. Declaration + Creation (combined)
        int[] arr2 = new int[5];

        // 3. Array literal (inline initialization)
        int[] arr3 = {1, 2, 3, 4, 5};  // Size inferred: 5

        // 4. Array literal with new (explicit)
        int[] arr4 = new int[]{1, 2, 3, 4, 5};

        // 5. Anonymous array (passed directly)
        printArray(new int[]{10, 20, 30});

        // Empty array
        int[] empty = new int[0];  // Valid, length = 0
        int[] empty2 = {};         // Also valid
    }

    static void printArray(int[] arr) {
        for (int i : arr) {
            System.out.print(i + " ");
        }
    }
}
```

### Default Values

```java
public class ArrayDefaultValues {
    public static void main(String[] args) {
        // Numeric types default to 0
        int[] intArr = new int[3];      // [0, 0, 0]
        double[] dblArr = new double[3]; // [0.0, 0.0, 0.0]

        // Boolean defaults to false
        boolean[] boolArr = new boolean[3]; // [false, false, false]

        // char defaults to '\u0000' (null character)
        char[] charArr = new char[3];   // ['\u0000', '\u0000', '\u0000']

        // Objects default to null
        String[] strArr = new String[3]; // [null, null, null]
        Integer[] intObjArr = new Integer[3]; // [null, null, null]
    }
}
```

---

## Array Operations

### Accessing Elements

```java
public class ArrayAccess {
    public static void main(String[] args) {
        int[] arr = {10, 20, 30, 40, 50};

        // Get element
        int first = arr[0];   // 10
        int last = arr[arr.length - 1];  // 50

        // Set element
        arr[2] = 100;  // [10, 20, 100, 40, 50]

        // ArrayIndexOutOfBoundsException
        try {
            int invalid = arr[10];  // Index out of bounds
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Invalid index!");
        }

        // Negative index
        try {
            int invalid = arr[-1];  // Also invalid
        } catch (ArrayIndexOutOfBoundsException e) {
            System.out.println("Negative index not allowed!");
        }
    }
}
```

### Iterating Over Arrays

```java
public class ArrayIteration {
    public static void main(String[] args) {
        String[] fruits = {"Apple", "Banana", "Cherry"};

        // 1. Traditional for loop
        for (int i = 0; i < fruits.length; i++) {
            System.out.println(i + ": " + fruits[i]);
        }

        // 2. Enhanced for loop (for-each)
        for (String fruit : fruits) {
            System.out.println(fruit);
        }

        // 3. While loop
        int i = 0;
        while (i < fruits.length) {
            System.out.println(fruits[i]);
            i++;
        }

        // 4. Using Arrays.stream() (Java 8+)
        Arrays.stream(fruits).forEach(System.out::println);

        // 5. Using IntStream for index
        IntStream.range(0, fruits.length)
                 .forEach(idx -> System.out.println(idx + ": " + fruits[idx]));

        // Reverse iteration
        for (int j = fruits.length - 1; j >= 0; j--) {
            System.out.println(fruits[j]);
        }
    }
}
```

### Array Length

```java
public class ArrayLength {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5};

        // length is a field, not a method
        int size = arr.length;  // 5 (not arr.length())

        // String uses .length() method
        String str = "Hello";
        int strLen = str.length();  // 5 (method call)

        // Collection uses .size() method
        List<Integer> list = Arrays.asList(1, 2, 3);
        int listSize = list.size();  // 3

        // Note: length is final, cannot change
        // arr.length = 10;  // Compilation error
    }
}
```

---

## Multi-Dimensional Arrays

### 2D Arrays

```java
public class TwoDArrays {
    public static void main(String[] args) {
        // Declaration and creation
        int[][] matrix = new int[3][4];  // 3 rows, 4 columns

        // Initialize with values
        int[][] matrix2 = {
            {1, 2, 3, 4},
            {5, 6, 7, 8},
            {9, 10, 11, 12}
        };

        // Access elements
        int element = matrix2[1][2];  // Row 1, Column 2 = 7

        // Modify elements
        matrix2[0][0] = 100;

        // Get dimensions
        int rows = matrix2.length;        // 3
        int cols = matrix2[0].length;     // 4

        // Iterate over 2D array
        for (int i = 0; i < matrix2.length; i++) {
            for (int j = 0; j < matrix2[i].length; j++) {
                System.out.print(matrix2[i][j] + " ");
            }
            System.out.println();
        }

        // Enhanced for loop
        for (int[] row : matrix2) {
            for (int val : row) {
                System.out.print(val + " ");
            }
            System.out.println();
        }
    }
}
```

### Jagged Arrays (Irregular)

```java
public class JaggedArrays {
    public static void main(String[] args) {
        // Rows can have different lengths
        int[][] jagged = new int[3][];
        jagged[0] = new int[2];  // First row: 2 columns
        jagged[1] = new int[4];  // Second row: 4 columns
        jagged[2] = new int[3];  // Third row: 3 columns

        // Initialize with literal
        int[][] jagged2 = {
            {1, 2},
            {3, 4, 5, 6},
            {7, 8, 9}
        };

        // Memory layout:
        // jagged -> [ref0, ref1, ref2]
        //            |      |     |
        //            v      v     v
        //           [1,2] [3,4,5,6] [7,8,9]

        // Iterate safely
        for (int i = 0; i < jagged2.length; i++) {
            for (int j = 0; j < jagged2[i].length; j++) {
                System.out.print(jagged2[i][j] + " ");
            }
            System.out.println();
        }
    }
}
```

### 3D and Higher Dimensional Arrays

```java
public class ThreeDArrays {
    public static void main(String[] args) {
        // 3D array: 2 x 3 x 4
        int[][][] cube = new int[2][3][4];

        // Think of it as:
        // 2 "planes"
        // Each plane has 3 rows
        // Each row has 4 columns

        // Initialize
        int[][][] cube2 = {
            {
                {1, 2, 3, 4},
                {5, 6, 7, 8},
                {9, 10, 11, 12}
            },
            {
                {13, 14, 15, 16},
                {17, 18, 19, 20},
                {21, 22, 23, 24}
            }
        };

        // Access element
        int element = cube2[1][2][3];  // 24

        // Iterate
        for (int[][] plane : cube2) {
            for (int[] row : plane) {
                for (int val : row) {
                    System.out.print(val + " ");
                }
                System.out.println();
            }
            System.out.println("---");
        }
    }
}
```

---

## Arrays Utility Class

### Common Arrays Methods

```java
import java.util.Arrays;

public class ArraysUtilityDemo {
    public static void main(String[] args) {
        int[] arr = {5, 2, 8, 1, 9, 3};

        // 1. toString() - String representation
        System.out.println(Arrays.toString(arr));  // [5, 2, 8, 1, 9, 3]

        // 2. sort() - Sorts in ascending order
        Arrays.sort(arr);
        System.out.println(Arrays.toString(arr));  // [1, 2, 3, 5, 8, 9]

        // Sort range
        int[] arr2 = {5, 2, 8, 1, 9, 3};
        Arrays.sort(arr2, 1, 4);  // Sort index 1 to 3
        System.out.println(Arrays.toString(arr2));  // [5, 1, 2, 8, 9, 3]

        // 3. binarySearch() - Search in sorted array
        int index = Arrays.binarySearch(arr, 5);  // 3
        int notFound = Arrays.binarySearch(arr, 10);  // Negative (insertion point)

        // 4. equals() - Compare arrays
        int[] a = {1, 2, 3};
        int[] b = {1, 2, 3};
        System.out.println(Arrays.equals(a, b));  // true
        System.out.println(a == b);  // false (different objects)

        // 5. fill() - Fill with value
        int[] filled = new int[5];
        Arrays.fill(filled, 42);
        System.out.println(Arrays.toString(filled));  // [42, 42, 42, 42, 42]

        // Fill range
        Arrays.fill(filled, 1, 4, 0);  // Fill index 1-3 with 0
        System.out.println(Arrays.toString(filled));  // [42, 0, 0, 0, 42]

        // 6. copyOf() - Create copy with new length
        int[] original = {1, 2, 3};
        int[] longer = Arrays.copyOf(original, 5);  // [1, 2, 3, 0, 0]
        int[] shorter = Arrays.copyOf(original, 2); // [1, 2]

        // 7. copyOfRange() - Copy range
        int[] range = Arrays.copyOfRange(original, 1, 3);  // [2, 3]

        // 8. asList() - Convert to List
        String[] strArr = {"A", "B", "C"};
        List<String> list = Arrays.asList(strArr);
        // Note: Fixed-size list backed by array
        // list.add("D");  // UnsupportedOperationException

        // 9. compare() - Lexicographic comparison (Java 9+)
        int result = Arrays.compare(new int[]{1, 2}, new int[]{1, 3});  // -1

        // 10. mismatch() - Find first difference (Java 9+)
        int mismatchIndex = Arrays.mismatch(
            new int[]{1, 2, 3, 4},
            new int[]{1, 2, 5, 4}
        );  // 2
    }
}
```

### Sorting Arrays

```java
public class ArraySorting {
    public static void main(String[] args) {
        // Primitive array - ascending
        int[] nums = {5, 2, 8, 1, 9};
        Arrays.sort(nums);  // [1, 2, 5, 8, 9]

        // Object array - natural order
        String[] names = {"Charlie", "Alice", "Bob"};
        Arrays.sort(names);  // [Alice, Bob, Charlie]

        // Custom comparator
        String[] names2 = {"Charlie", "Alice", "Bob"};
        Arrays.sort(names2, (a, b) -> b.compareTo(a));  // Descending
        // [Charlie, Bob, Alice]

        // Sort by length
        String[] words = {"Apple", "Hi", "Banana", "Cat"};
        Arrays.sort(words, Comparator.comparingInt(String::length));
        // [Hi, Cat, Apple, Banana]

        // Descending order for primitives (workaround)
        int[] nums2 = {5, 2, 8, 1, 9};
        // Method 1: Sort and reverse
        Arrays.sort(nums2);
        reverse(nums2);

        // Method 2: Use Integer array
        Integer[] numsObj = {5, 2, 8, 1, 9};
        Arrays.sort(numsObj, Collections.reverseOrder());
        // [9, 8, 5, 2, 1]
    }

    static void reverse(int[] arr) {
        for (int i = 0; i < arr.length / 2; i++) {
            int temp = arr[i];
            arr[i] = arr[arr.length - 1 - i];
            arr[arr.length - 1 - i] = temp;
        }
    }
}
```

### Parallel Operations (Java 8+)

```java
public class ParallelArrayOperations {
    public static void main(String[] args) {
        int[] arr = new int[10000000];
        Arrays.fill(arr, 1);

        // Parallel sort (for large arrays)
        Arrays.parallelSort(arr);

        // Parallel prefix (cumulative operation)
        int[] nums = {1, 2, 3, 4, 5};
        Arrays.parallelPrefix(nums, (a, b) -> a + b);
        // [1, 3, 6, 10, 15] - running sum

        // Parallel setAll
        int[] generated = new int[10];
        Arrays.parallelSetAll(generated, i -> i * i);
        // [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

        // setAll (non-parallel version)
        Arrays.setAll(generated, i -> i * 2);
        // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
    }
}
```

---

## Array Copying

### Different Copy Methods

```java
public class ArrayCopying {
    public static void main(String[] args) {
        int[] original = {1, 2, 3, 4, 5};

        // 1. Assignment (NOT a copy - same reference!)
        int[] notACopy = original;
        notACopy[0] = 100;
        System.out.println(original[0]);  // 100 - original also changed!
        original[0] = 1;  // Reset

        // 2. Arrays.copyOf()
        int[] copy1 = Arrays.copyOf(original, original.length);
        copy1[0] = 100;
        System.out.println(original[0]);  // 1 - original unchanged

        // 3. System.arraycopy() - Fastest
        int[] copy2 = new int[original.length];
        System.arraycopy(original, 0, copy2, 0, original.length);
        // (src, srcPos, dest, destPos, length)

        // 4. clone()
        int[] copy3 = original.clone();

        // 5. Manual copy
        int[] copy4 = new int[original.length];
        for (int i = 0; i < original.length; i++) {
            copy4[i] = original[i];
        }

        // 6. Stream (Java 8+)
        int[] copy5 = Arrays.stream(original).toArray();
    }
}
```

### Shallow Copy vs Deep Copy

```java
public class ShallowVsDeepCopy {
    public static void main(String[] args) {
        // For primitive arrays, all copies are "deep" (values copied)
        int[] original = {1, 2, 3};
        int[] copy = original.clone();
        copy[0] = 100;
        System.out.println(original[0]);  // 1 (unchanged)

        // For object arrays, copies are SHALLOW (references copied)
        StringBuilder[] sbArr = {
            new StringBuilder("A"),
            new StringBuilder("B")
        };
        StringBuilder[] shallowCopy = sbArr.clone();
        shallowCopy[0].append("X");  // Modifies the same object!
        System.out.println(sbArr[0]);  // "AX" - original affected!

        // Deep copy for object arrays
        StringBuilder[] deepCopy = new StringBuilder[sbArr.length];
        for (int i = 0; i < sbArr.length; i++) {
            deepCopy[i] = new StringBuilder(sbArr[i]);
        }
        deepCopy[0].append("Y");
        System.out.println(sbArr[0]);  // "AX" - original unchanged
    }
}
```

### 2D Array Copying

```java
public class TwoDArrayCopying {
    public static void main(String[] args) {
        int[][] original = {{1, 2}, {3, 4}};

        // Shallow copy - only first dimension copied
        int[][] shallowCopy = original.clone();
        shallowCopy[0][0] = 100;
        System.out.println(original[0][0]);  // 100 - affected!

        // Deep copy
        int[][] deepCopy = new int[original.length][];
        for (int i = 0; i < original.length; i++) {
            deepCopy[i] = original[i].clone();
        }
        // OR
        int[][] deepCopy2 = Arrays.stream(original)
                                  .map(int[]::clone)
                                  .toArray(int[][]::new);
    }
}
```

---

## Common Array Algorithms

### Searching

```java
public class ArraySearching {
    // Linear Search - O(n)
    public static int linearSearch(int[] arr, int target) {
        for (int i = 0; i < arr.length; i++) {
            if (arr[i] == target) {
                return i;
            }
        }
        return -1;
    }

    // Binary Search - O(log n) - requires sorted array
    public static int binarySearch(int[] arr, int target) {
        int left = 0, right = arr.length - 1;
        while (left <= right) {
            int mid = left + (right - left) / 2;  // Avoid overflow
            if (arr[mid] == target) {
                return mid;
            } else if (arr[mid] < target) {
                left = mid + 1;
            } else {
                right = mid - 1;
            }
        }
        return -1;
    }

    public static void main(String[] args) {
        int[] arr = {1, 3, 5, 7, 9, 11, 13};
        System.out.println(linearSearch(arr, 7));  // 3
        System.out.println(binarySearch(arr, 7));  // 3
    }
}
```

### Finding Min/Max

```java
public class MinMaxArray {
    // Using loop
    public static int findMax(int[] arr) {
        if (arr == null || arr.length == 0) {
            throw new IllegalArgumentException("Array is empty");
        }
        int max = arr[0];
        for (int i = 1; i < arr.length; i++) {
            if (arr[i] > max) {
                max = arr[i];
            }
        }
        return max;
    }

    // Using Stream
    public static int findMaxStream(int[] arr) {
        return Arrays.stream(arr).max().orElseThrow();
    }

    // Find both min and max in single pass
    public static int[] findMinMax(int[] arr) {
        int min = arr[0], max = arr[0];
        for (int val : arr) {
            if (val < min) min = val;
            if (val > max) max = val;
        }
        return new int[]{min, max};
    }

    // Find second largest
    public static int secondLargest(int[] arr) {
        int first = Integer.MIN_VALUE;
        int second = Integer.MIN_VALUE;

        for (int val : arr) {
            if (val > first) {
                second = first;
                first = val;
            } else if (val > second && val != first) {
                second = val;
            }
        }
        return second;
    }
}
```

### Reversing Array

```java
public class ArrayReverse {
    // In-place reverse
    public static void reverse(int[] arr) {
        int left = 0, right = arr.length - 1;
        while (left < right) {
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            left++;
            right--;
        }
    }

    // Create new reversed array
    public static int[] reverseCopy(int[] arr) {
        int[] reversed = new int[arr.length];
        for (int i = 0; i < arr.length; i++) {
            reversed[i] = arr[arr.length - 1 - i];
        }
        return reversed;
    }

    // Using Collections (for Integer array)
    public static void reverseWithCollections(Integer[] arr) {
        Collections.reverse(Arrays.asList(arr));
    }
}
```

### Removing Duplicates

```java
public class RemoveDuplicates {
    // From sorted array (in-place) - O(n)
    public static int removeDuplicatesSorted(int[] arr) {
        if (arr.length == 0) return 0;
        int writeIndex = 1;
        for (int i = 1; i < arr.length; i++) {
            if (arr[i] != arr[writeIndex - 1]) {
                arr[writeIndex] = arr[i];
                writeIndex++;
            }
        }
        return writeIndex;  // New length
    }

    // From unsorted array - O(n) with extra space
    public static int[] removeDuplicatesUnsorted(int[] arr) {
        return Arrays.stream(arr).distinct().toArray();
    }

    // Using Set
    public static Integer[] removeDuplicatesSet(Integer[] arr) {
        Set<Integer> set = new LinkedHashSet<>(Arrays.asList(arr));
        return set.toArray(new Integer[0]);
    }
}
```

### Rotating Array

```java
public class ArrayRotation {
    // Rotate left by k positions
    public static void rotateLeft(int[] arr, int k) {
        k = k % arr.length;  // Handle k > length
        reverse(arr, 0, k - 1);
        reverse(arr, k, arr.length - 1);
        reverse(arr, 0, arr.length - 1);
    }

    // Rotate right by k positions
    public static void rotateRight(int[] arr, int k) {
        k = k % arr.length;
        reverse(arr, 0, arr.length - 1);
        reverse(arr, 0, k - 1);
        reverse(arr, k, arr.length - 1);
    }

    private static void reverse(int[] arr, int start, int end) {
        while (start < end) {
            int temp = arr[start];
            arr[start] = arr[end];
            arr[end] = temp;
            start++;
            end--;
        }
    }

    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5};
        rotateRight(arr, 2);
        System.out.println(Arrays.toString(arr));  // [4, 5, 1, 2, 3]
    }
}
```

---

## Arrays vs ArrayList

### Comparison Table

| Feature           | Array                         | ArrayList                |
| ----------------- | ----------------------------- | ------------------------ |
| Size              | Fixed                         | Dynamic                  |
| Type              | Primitives + Objects          | Objects only             |
| Performance       | Faster                        | Slightly slower          |
| Memory            | Less overhead                 | More overhead            |
| Type Safety       | Less (with generics: similar) | Generic support          |
| Null Elements     | Allowed                       | Allowed                  |
| Multi-dimensional | Supported                     | Nested lists             |
| Methods           | Few (length, clone)           | Many (add, remove, etc.) |

### When to Use What

```java
public class ArrayVsArrayList {
    public static void main(String[] args) {
        // Use Array when:
        // 1. Size is known and fixed
        int[] fixedSize = new int[100];

        // 2. Working with primitives (better performance)
        int[] primitives = {1, 2, 3, 4, 5};

        // 3. Multi-dimensional data
        int[][] matrix = new int[10][10];

        // 4. Performance is critical
        // Arrays are faster for access and iteration

        // Use ArrayList when:
        // 1. Size needs to change
        List<Integer> dynamic = new ArrayList<>();
        dynamic.add(1);
        dynamic.remove(0);

        // 2. Need rich API
        dynamic.contains(5);
        dynamic.indexOf(3);
        Collections.sort(dynamic);

        // 3. Working with APIs that expect List
        void processData(List<String> data);

        // 4. Need thread-safe version easily
        List<Integer> syncList = Collections.synchronizedList(new ArrayList<>());
    }
}
```

### Converting Between Array and ArrayList

```java
public class ArrayListConversion {
    public static void main(String[] args) {
        // Array to ArrayList
        String[] arr = {"A", "B", "C"};

        // Method 1: Arrays.asList (fixed-size list!)
        List<String> fixedList = Arrays.asList(arr);
        // fixedList.add("D");  // UnsupportedOperationException

        // Method 2: New ArrayList (modifiable)
        List<String> modifiableList = new ArrayList<>(Arrays.asList(arr));
        modifiableList.add("D");  // OK

        // Method 3: Stream (Java 8+)
        List<String> streamList = Arrays.stream(arr)
                                        .collect(Collectors.toList());

        // Primitive array to ArrayList
        int[] intArr = {1, 2, 3};
        List<Integer> intList = Arrays.stream(intArr)
                                      .boxed()
                                      .collect(Collectors.toList());

        // ArrayList to Array
        List<String> list = new ArrayList<>(Arrays.asList("X", "Y", "Z"));

        // Method 1: toArray() with type
        String[] newArr = list.toArray(new String[0]);

        // Method 2: toArray() with generator (Java 11+)
        String[] newArr2 = list.toArray(String[]::new);

        // Primitive ArrayList to array
        List<Integer> integerList = Arrays.asList(1, 2, 3);
        int[] primitiveArr = integerList.stream()
                                        .mapToInt(Integer::intValue)
                                        .toArray();
    }
}
```

---

## Interview Questions

### Q1: What is the difference between array.length and String.length()?

**Answer:**

- `array.length` is a **field** (property)
- `String.length()` is a **method**

```java
int[] arr = {1, 2, 3};
int arrayLength = arr.length;      // Field access

String str = "Hello";
int stringLength = str.length();   // Method call
```

Arrays have `length` as a final field because array size is fixed at creation and immutable.

---

### Q2: Can we change array size after creation?

**Answer:**
**No.** Arrays have fixed size. To "resize," you must create a new array and copy elements.

```java
int[] original = {1, 2, 3};
// Cannot do: original.length = 5;

// Instead, create new array:
int[] bigger = Arrays.copyOf(original, 5);
// Or use ArrayList for dynamic sizing
```

---

### Q3: What is the output?

```java
int[] arr1 = {1, 2, 3};
int[] arr2 = {1, 2, 3};
System.out.println(arr1 == arr2);
System.out.println(arr1.equals(arr2));
System.out.println(Arrays.equals(arr1, arr2));
```

**Answer:**

```
false    // Different objects (reference comparison)
false    // Arrays inherit Object.equals (same as ==)
true     // Arrays.equals compares content
```

---

### Q4: How to print array contents properly?

**Answer:**

```java
int[] arr = {1, 2, 3};

// Wrong way (prints object reference)
System.out.println(arr);  // [I@hashcode

// Correct ways:
System.out.println(Arrays.toString(arr));      // [1, 2, 3]

// For 2D arrays:
int[][] matrix = {{1, 2}, {3, 4}};
System.out.println(Arrays.deepToString(matrix));  // [[1, 2], [3, 4]]
```

---

### Q5: What is ArrayStoreException?

**Answer:**
Thrown when trying to store wrong type in an array.

```java
Object[] objArr = new String[3];  // Legal - covariance
objArr[0] = "Hello";              // OK
objArr[1] = 123;                  // ArrayStoreException at runtime!
```

This happens because of array covariance - a `String[]` can be assigned to `Object[]`, but you can only store Strings in it.

---

### Q6: Difference between System.arraycopy() and Arrays.copyOf()?

**Answer:**

| System.arraycopy()         | Arrays.copyOf()                  |
| -------------------------- | -------------------------------- |
| Copies to existing array   | Creates new array                |
| Requires destination array | Returns new array                |
| Can copy partial array     | Copies from start                |
| Native method (faster)     | Uses System.arraycopy internally |

```java
int[] src = {1, 2, 3, 4, 5};

// System.arraycopy - need existing destination
int[] dest = new int[3];
System.arraycopy(src, 1, dest, 0, 3);  // [2, 3, 4]

// Arrays.copyOf - creates new array
int[] copy = Arrays.copyOf(src, 3);  // [1, 2, 3]
```

---

### Q7: How to find duplicate elements in array?

```java
public static Set<Integer> findDuplicates(int[] arr) {
    Set<Integer> seen = new HashSet<>();
    Set<Integer> duplicates = new HashSet<>();

    for (int num : arr) {
        if (!seen.add(num)) {  // add returns false if already exists
            duplicates.add(num);
        }
    }
    return duplicates;
}
```

---

### Q8: What is the output?

```java
public class Test {
    public static void main(String[] args) {
        int[] arr = {1, 2, 3};
        modify(arr);
        System.out.println(Arrays.toString(arr));

        arr = new int[]{4, 5, 6};
        reassign(arr);
        System.out.println(Arrays.toString(arr));
    }

    static void modify(int[] a) {
        a[0] = 100;
    }

    static void reassign(int[] a) {
        a = new int[]{7, 8, 9};
    }
}
```

**Answer:**

```
[100, 2, 3]    // modify changed original array content
[4, 5, 6]     // reassign only changed local reference
```

Arrays are passed by **value of reference**. Modifying content affects original; reassigning reference doesn't.

---

### Q9: How to create immutable array?

**Answer:**
Arrays are mutable by nature. Workarounds:

```java
// 1. Return defensive copy
public int[] getData() {
    return Arrays.copyOf(data, data.length);
}

// 2. Use Collections.unmodifiableList
public List<Integer> getData() {
    return Collections.unmodifiableList(Arrays.asList(data));
}

// 3. Use List.of() (Java 9+)
List<Integer> immutable = List.of(1, 2, 3);
// immutable.add(4);  // UnsupportedOperationException
```

---

### Q10: Find missing number in array 1 to n

```java
// Array contains n-1 elements from 1 to n, find missing
public static int findMissing(int[] arr, int n) {
    // Method 1: Sum formula
    int expectedSum = n * (n + 1) / 2;
    int actualSum = 0;
    for (int num : arr) {
        actualSum += num;
    }
    return expectedSum - actualSum;

    // Method 2: XOR
    int xor = 0;
    for (int i = 1; i <= n; i++) {
        xor ^= i;
    }
    for (int num : arr) {
        xor ^= num;
    }
    return xor;  // Missing number
}
```

---

## Common Interview Traps

### Trap 1: "Arrays implement Collection interface"

**No!** Arrays are not part of Collections framework. They're fundamental language feature.

### Trap 2: "int[] arr = new int[5] creates 5 int objects"

**No!** Creates 1 array object containing 5 primitive int values.

### Trap 3: "Arrays.asList() returns ArrayList"

Returns `Arrays.ArrayList` (inner class), not `java.util.ArrayList`. It's fixed-size!

### Trap 4: "Array index can be long"

**No!** Array index must be int. Max array size is Integer.MAX_VALUE - 2.

### Trap 5: "Clone creates deep copy for object arrays"

**No!** Clone creates shallow copy - object references are copied, not objects.

---

## Key Takeaways

1. **Arrays are objects** - stored in heap, even for primitives
2. **Fixed size** - cannot resize, create new array instead
3. **Zero-indexed** - first element at index 0
4. **Contiguous memory** - fast access O(1)
5. **Use Arrays class** - for utility methods (sort, search, copy)
6. **Clone is shallow** - for object arrays
7. **Covariance** - can cause ArrayStoreException
8. **Use ArrayList** - when size needs to change
9. **Prefer Arrays.copyOf** - over manual copying
10. **Arrays.toString()** - for proper string representation
