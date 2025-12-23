# Stream API Coding Problems - Complete Interview Guide

## Table of Contents

1. [Basic Stream Operations](#basic-stream-operations)
2. [Filtering and Searching](#filtering-and-searching)
3. [Aggregation Problems](#aggregation-problems)
4. [Grouping and Partitioning](#grouping-and-partitioning)
5. [String Manipulation with Streams](#string-manipulation-with-streams)
6. [Sorting and Comparison](#sorting-and-comparison)
7. [FlatMap and Nested Collections](#flatmap-and-nested-collections)
8. [Reduction Operations](#reduction-operations)
9. [Parallel Streams](#parallel-streams)
10. [Real-World Scenarios](#real-world-scenarios)
11. [Interview Questions & Answers](#interview-questions--answers)

---

## Setup: Common Classes Used

```java
@Data
@AllArgsConstructor
class Employee {
    private Long id;
    private String name;
    private String department;
    private double salary;
    private int age;
    private String gender;
    private LocalDate joiningDate;
}

@Data
@AllArgsConstructor
class Transaction {
    private Long id;
    private String type;      // "CREDIT" or "DEBIT"
    private double amount;
    private LocalDate date;
    private String category;
}

@Data
@AllArgsConstructor
class Order {
    private Long orderId;
    private String customerId;
    private List<OrderItem> items;
    private LocalDate orderDate;
}

@Data
@AllArgsConstructor
class OrderItem {
    private String productName;
    private int quantity;
    private double price;
}
```

---

## Basic Stream Operations

### Problem 1: Find All Distinct Elements

```java
// Given a list of integers with duplicates, find distinct elements
public class DistinctElements {

    public List<Integer> findDistinct(List<Integer> numbers) {
        return numbers.stream()
            .distinct()
            .collect(Collectors.toList());
    }

    // With custom objects - need equals() and hashCode()
    public List<Employee> findDistinctEmployees(List<Employee> employees) {
        return employees.stream()
            .distinct()
            .collect(Collectors.toList());
    }

    // Distinct by specific field
    public List<Employee> distinctByDepartment(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.collectingAndThen(
                Collectors.toMap(
                    Employee::getDepartment,
                    e -> e,
                    (e1, e2) -> e1
                ),
                m -> new ArrayList<>(m.values())
            ));
    }
}
```

### Problem 2: Find Duplicate Elements

```java
public class FindDuplicates {

    // Method 1: Using Set
    public List<Integer> findDuplicates(List<Integer> numbers) {
        Set<Integer> seen = new HashSet<>();
        return numbers.stream()
            .filter(n -> !seen.add(n))  // add() returns false if already exists
            .distinct()
            .collect(Collectors.toList());
    }

    // Method 2: Using groupingBy and counting
    public List<Integer> findDuplicatesWithGrouping(List<Integer> numbers) {
        return numbers.stream()
            .collect(Collectors.groupingBy(n -> n, Collectors.counting()))
            .entrySet().stream()
            .filter(entry -> entry.getValue() > 1)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }

    // Find duplicate employees by name
    public List<String> findDuplicateNames(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(Employee::getName, Collectors.counting()))
            .entrySet().stream()
            .filter(e -> e.getValue() > 1)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}
```

### Problem 3: Count Element Frequency

```java
public class FrequencyCounter {

    // Count frequency of each element
    public Map<Integer, Long> countFrequency(List<Integer> numbers) {
        return numbers.stream()
            .collect(Collectors.groupingBy(
                n -> n,
                Collectors.counting()
            ));
    }

    // Count frequency of characters in a string
    public Map<Character, Long> countCharFrequency(String str) {
        return str.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(
                c -> c,
                Collectors.counting()
            ));
    }

    // Find first non-repeating character
    public Optional<Character> firstNonRepeating(String str) {
        return str.chars()
            .mapToObj(c -> (char) c)
            .collect(Collectors.groupingBy(
                c -> c,
                LinkedHashMap::new,  // Maintain insertion order
                Collectors.counting()
            ))
            .entrySet().stream()
            .filter(e -> e.getValue() == 1)
            .map(Map.Entry::getKey)
            .findFirst();
    }
}
```

---

## Filtering and Searching

### Problem 4: Filter Employees by Multiple Criteria

```java
public class EmployeeFilter {

    // Find employees in IT department earning more than 50000
    public List<Employee> filterBySalaryAndDept(List<Employee> employees) {
        return employees.stream()
            .filter(e -> "IT".equals(e.getDepartment()))
            .filter(e -> e.getSalary() > 50000)
            .collect(Collectors.toList());
    }

    // Find employees joined in last 2 years
    public List<Employee> recentJoiners(List<Employee> employees) {
        LocalDate twoYearsAgo = LocalDate.now().minusYears(2);
        return employees.stream()
            .filter(e -> e.getJoiningDate().isAfter(twoYearsAgo))
            .collect(Collectors.toList());
    }

    // Dynamic filter using Predicate
    public List<Employee> filter(List<Employee> employees, Predicate<Employee> condition) {
        return employees.stream()
            .filter(condition)
            .collect(Collectors.toList());
    }

    // Usage
    public void demo(List<Employee> employees) {
        // Combining predicates
        Predicate<Employee> seniorEmployee = e -> e.getAge() > 40;
        Predicate<Employee> highEarner = e -> e.getSalary() > 100000;

        List<Employee> result = filter(employees, seniorEmployee.and(highEarner));
    }
}
```

### Problem 5: Find Nth Highest/Lowest Element

```java
public class NthElement {

    // Find second highest salary
    public Optional<Double> secondHighestSalary(List<Employee> employees) {
        return employees.stream()
            .map(Employee::getSalary)
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(1)
            .findFirst();
    }

    // Find Nth highest salary
    public Optional<Double> nthHighestSalary(List<Employee> employees, int n) {
        return employees.stream()
            .map(Employee::getSalary)
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(n - 1)
            .findFirst();
    }

    // Find employee with second highest salary
    public Optional<Employee> employeeWithSecondHighestSalary(List<Employee> employees) {
        return employees.stream()
            .map(Employee::getSalary)
            .distinct()
            .sorted(Comparator.reverseOrder())
            .skip(1)
            .findFirst()
            .flatMap(salary -> employees.stream()
                .filter(e -> e.getSalary() == salary)
                .findFirst()
            );
    }

    // Find third youngest employee
    public Optional<Employee> thirdYoungest(List<Employee> employees) {
        return employees.stream()
            .sorted(Comparator.comparingInt(Employee::getAge))
            .skip(2)
            .findFirst();
    }
}
```

### Problem 6: Check Conditions

```java
public class ConditionChecker {

    // Check if any employee earns more than 100000
    public boolean anyHighEarner(List<Employee> employees) {
        return employees.stream()
            .anyMatch(e -> e.getSalary() > 100000);
    }

    // Check if all employees are adults
    public boolean allAdults(List<Employee> employees) {
        return employees.stream()
            .allMatch(e -> e.getAge() >= 18);
    }

    // Check if no employee is from Marketing
    public boolean noMarketingEmployee(List<Employee> employees) {
        return employees.stream()
            .noneMatch(e -> "Marketing".equals(e.getDepartment()));
    }

    // Find first employee in IT
    public Optional<Employee> findFirstInIT(List<Employee> employees) {
        return employees.stream()
            .filter(e -> "IT".equals(e.getDepartment()))
            .findFirst();
    }

    // Find any employee (useful in parallel streams)
    public Optional<Employee> findAnyInIT(List<Employee> employees) {
        return employees.parallelStream()
            .filter(e -> "IT".equals(e.getDepartment()))
            .findAny();
    }
}
```

---

## Aggregation Problems

### Problem 7: Calculate Statistics

```java
public class StatisticsCalculator {

    // Calculate average salary
    public double averageSalary(List<Employee> employees) {
        return employees.stream()
            .mapToDouble(Employee::getSalary)
            .average()
            .orElse(0.0);
    }

    // Calculate sum of salaries
    public double totalSalary(List<Employee> employees) {
        return employees.stream()
            .mapToDouble(Employee::getSalary)
            .sum();
    }

    // Find min and max salary
    public DoubleSummaryStatistics salaryStats(List<Employee> employees) {
        return employees.stream()
            .mapToDouble(Employee::getSalary)
            .summaryStatistics();
    }

    // Usage of summary statistics
    public void printStats(List<Employee> employees) {
        DoubleSummaryStatistics stats = salaryStats(employees);
        System.out.println("Count: " + stats.getCount());
        System.out.println("Sum: " + stats.getSum());
        System.out.println("Min: " + stats.getMin());
        System.out.println("Max: " + stats.getMax());
        System.out.println("Average: " + stats.getAverage());
    }

    // Average salary by department
    public Map<String, Double> avgSalaryByDept(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
            ));
    }
}
```

### Problem 8: Find Max/Min with Objects

```java
public class MaxMinFinder {

    // Find employee with highest salary
    public Optional<Employee> highestPaidEmployee(List<Employee> employees) {
        return employees.stream()
            .max(Comparator.comparingDouble(Employee::getSalary));
    }

    // Find youngest employee
    public Optional<Employee> youngestEmployee(List<Employee> employees) {
        return employees.stream()
            .min(Comparator.comparingInt(Employee::getAge));
    }

    // Find highest paid in each department
    public Map<String, Optional<Employee>> highestPaidByDept(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.maxBy(Comparator.comparingDouble(Employee::getSalary))
            ));
    }

    // Find employee with max salary and earliest joining date (tie-breaker)
    public Optional<Employee> highestPaidEarliestJoiner(List<Employee> employees) {
        return employees.stream()
            .max(Comparator
                .comparingDouble(Employee::getSalary)
                .thenComparing(Employee::getJoiningDate, Comparator.reverseOrder())
            );
    }
}
```

---

## Grouping and Partitioning

### Problem 9: Group Employees

```java
public class EmployeeGrouping {

    // Group by department
    public Map<String, List<Employee>> groupByDept(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment));
    }

    // Group by department and count
    public Map<String, Long> countByDept(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
            ));
    }

    // Group by department and get names
    public Map<String, List<String>> namesByDept(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(Employee::getName, Collectors.toList())
            ));
    }

    // Multi-level grouping: Department -> Gender -> Employees
    public Map<String, Map<String, List<Employee>>> groupByDeptAndGender(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.groupingBy(Employee::getGender)
            ));
    }

    // Group by age range
    public Map<String, List<Employee>> groupByAgeRange(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(e -> {
                if (e.getAge() < 25) return "Young";
                else if (e.getAge() < 40) return "Middle";
                else return "Senior";
            }));
    }
}
```

### Problem 10: Partition Employees

```java
public class EmployeePartitioning {

    // Partition by salary threshold
    public Map<Boolean, List<Employee>> partitionBySalary(List<Employee> employees, double threshold) {
        return employees.stream()
            .collect(Collectors.partitioningBy(e -> e.getSalary() > threshold));
    }

    // Partition and count
    public Map<Boolean, Long> partitionAndCount(List<Employee> employees, double threshold) {
        return employees.stream()
            .collect(Collectors.partitioningBy(
                e -> e.getSalary() > threshold,
                Collectors.counting()
            ));
    }

    // Partition adults vs minors
    public Map<Boolean, List<String>> partitionByAge(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.partitioningBy(
                e -> e.getAge() >= 18,
                Collectors.mapping(Employee::getName, Collectors.toList())
            ));
    }
}
```

### Problem 11: Joining Strings

```java
public class StringJoining {

    // Join all employee names
    public String joinNames(List<Employee> employees) {
        return employees.stream()
            .map(Employee::getName)
            .collect(Collectors.joining(", "));
    }

    // Join with prefix and suffix
    public String joinNamesFormatted(List<Employee> employees) {
        return employees.stream()
            .map(Employee::getName)
            .collect(Collectors.joining(", ", "[", "]"));
    }

    // Join department wise
    public Map<String, String> namesByDeptJoined(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.mapping(
                    Employee::getName,
                    Collectors.joining(", ")
                )
            ));
    }
}
```

---

## String Manipulation with Streams

### Problem 12: String Processing

```java
public class StringStreamProblems {

    // Count words in a sentence
    public long countWords(String sentence) {
        return Arrays.stream(sentence.split("\\s+"))
            .filter(word -> !word.isEmpty())
            .count();
    }

    // Find longest word
    public Optional<String> longestWord(String sentence) {
        return Arrays.stream(sentence.split("\\s+"))
            .max(Comparator.comparingInt(String::length));
    }

    // Count vowels in a string
    public long countVowels(String str) {
        return str.toLowerCase().chars()
            .filter(c -> "aeiou".indexOf(c) != -1)
            .count();
    }

    // Reverse each word in a sentence
    public String reverseWords(String sentence) {
        return Arrays.stream(sentence.split("\\s+"))
            .map(word -> new StringBuilder(word).reverse().toString())
            .collect(Collectors.joining(" "));
    }

    // Find all unique words (case-insensitive)
    public Set<String> uniqueWords(String sentence) {
        return Arrays.stream(sentence.split("\\s+"))
            .map(String::toLowerCase)
            .collect(Collectors.toSet());
    }

    // Word frequency map
    public Map<String, Long> wordFrequency(String sentence) {
        return Arrays.stream(sentence.toLowerCase().split("\\s+"))
            .filter(word -> !word.isEmpty())
            .collect(Collectors.groupingBy(
                word -> word,
                Collectors.counting()
            ));
    }

    // Find palindrome words
    public List<String> palindromeWords(String sentence) {
        return Arrays.stream(sentence.split("\\s+"))
            .filter(word -> {
                String reversed = new StringBuilder(word).reverse().toString();
                return word.equalsIgnoreCase(reversed) && word.length() > 1;
            })
            .collect(Collectors.toList());
    }
}
```

### Problem 13: Anagram Detection

```java
public class AnagramStream {

    // Check if two strings are anagrams
    public boolean areAnagrams(String s1, String s2) {
        if (s1.length() != s2.length()) return false;

        String sorted1 = s1.toLowerCase().chars()
            .sorted()
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());

        String sorted2 = s2.toLowerCase().chars()
            .sorted()
            .mapToObj(c -> String.valueOf((char) c))
            .collect(Collectors.joining());

        return sorted1.equals(sorted2);
    }

    // Group anagrams together
    public List<List<String>> groupAnagrams(List<String> words) {
        return new ArrayList<>(words.stream()
            .collect(Collectors.groupingBy(word -> {
                return word.toLowerCase().chars()
                    .sorted()
                    .mapToObj(c -> String.valueOf((char) c))
                    .collect(Collectors.joining());
            }))
            .values());
    }
}
```

---

## Sorting and Comparison

### Problem 14: Multiple Sort Criteria

```java
public class MultiSorting {

    // Sort by department, then by salary descending, then by name
    public List<Employee> multiSort(List<Employee> employees) {
        return employees.stream()
            .sorted(Comparator
                .comparing(Employee::getDepartment)
                .thenComparing(Employee::getSalary, Comparator.reverseOrder())
                .thenComparing(Employee::getName))
            .collect(Collectors.toList());
    }

    // Sort by age and get top 5 youngest
    public List<Employee> top5Youngest(List<Employee> employees) {
        return employees.stream()
            .sorted(Comparator.comparingInt(Employee::getAge))
            .limit(5)
            .collect(Collectors.toList());
    }

    // Sort by salary and get bottom 3
    public List<Employee> bottom3LowestPaid(List<Employee> employees) {
        return employees.stream()
            .sorted(Comparator.comparingDouble(Employee::getSalary))
            .limit(3)
            .collect(Collectors.toList());
    }

    // Custom sorting with null handling
    public List<Employee> sortWithNulls(List<Employee> employees) {
        return employees.stream()
            .sorted(Comparator.comparing(
                Employee::getDepartment,
                Comparator.nullsLast(Comparator.naturalOrder())
            ))
            .collect(Collectors.toList());
    }
}
```

### Problem 15: Top N Elements

```java
public class TopNElements {

    // Top 3 highest paid employees
    public List<Employee> top3HighestPaid(List<Employee> employees) {
        return employees.stream()
            .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
            .limit(3)
            .collect(Collectors.toList());
    }

    // Top 3 highest paid per department
    public Map<String, List<Employee>> top3PerDepartment(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                    Collectors.toList(),
                    list -> list.stream()
                        .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
                        .limit(3)
                        .collect(Collectors.toList())
                )
            ));
    }

    // Skip first 2 and get next 3
    public List<Employee> getPage(List<Employee> employees, int page, int size) {
        return employees.stream()
            .skip((long) page * size)
            .limit(size)
            .collect(Collectors.toList());
    }
}
```

---

## FlatMap and Nested Collections

### Problem 16: Flatten Nested Collections

```java
public class FlatMapProblems {

    // Flatten list of lists
    public List<Integer> flattenList(List<List<Integer>> nestedList) {
        return nestedList.stream()
            .flatMap(List::stream)
            .collect(Collectors.toList());
    }

    // Get all order items from all orders
    public List<OrderItem> getAllItems(List<Order> orders) {
        return orders.stream()
            .flatMap(order -> order.getItems().stream())
            .collect(Collectors.toList());
    }

    // Get all unique product names
    public Set<String> allProductNames(List<Order> orders) {
        return orders.stream()
            .flatMap(order -> order.getItems().stream())
            .map(OrderItem::getProductName)
            .collect(Collectors.toSet());
    }

    // Calculate total value of all orders
    public double totalOrderValue(List<Order> orders) {
        return orders.stream()
            .flatMap(order -> order.getItems().stream())
            .mapToDouble(item -> item.getQuantity() * item.getPrice())
            .sum();
    }

    // Find orders containing a specific product
    public List<Order> ordersContaining(List<Order> orders, String productName) {
        return orders.stream()
            .filter(order -> order.getItems().stream()
                .anyMatch(item -> productName.equals(item.getProductName())))
            .collect(Collectors.toList());
    }
}
```

### Problem 17: Map of Lists Operations

```java
public class MapOfListsProblems {

    // Flatten Map<String, List<String>> to List<String>
    public List<String> flattenMapValues(Map<String, List<String>> map) {
        return map.values().stream()
            .flatMap(List::stream)
            .collect(Collectors.toList());
    }

    // Get all unique values from Map<String, List<String>>
    public Set<String> uniqueMapValues(Map<String, List<String>> map) {
        return map.values().stream()
            .flatMap(List::stream)
            .collect(Collectors.toSet());
    }

    // Invert Map<String, List<String>> to Map<String, String>
    public Map<String, String> invertMap(Map<String, List<String>> map) {
        return map.entrySet().stream()
            .flatMap(entry -> entry.getValue().stream()
                .map(value -> Map.entry(value, entry.getKey())))
            .collect(Collectors.toMap(
                Map.Entry::getKey,
                Map.Entry::getValue,
                (v1, v2) -> v1  // Handle duplicates
            ));
    }
}
```

---

## Reduction Operations

### Problem 18: Custom Reduce Operations

```java
public class ReduceProblems {

    // Sum of all numbers
    public int sum(List<Integer> numbers) {
        return numbers.stream()
            .reduce(0, Integer::sum);
    }

    // Product of all numbers
    public int product(List<Integer> numbers) {
        return numbers.stream()
            .reduce(1, (a, b) -> a * b);
    }

    // Find maximum
    public Optional<Integer> findMax(List<Integer> numbers) {
        return numbers.stream()
            .reduce(Integer::max);
    }

    // Concatenate all strings
    public String concatenate(List<String> strings) {
        return strings.stream()
            .reduce("", String::concat);
    }

    // Calculate total salary using reduce
    public double totalSalaryWithReduce(List<Employee> employees) {
        return employees.stream()
            .map(Employee::getSalary)
            .reduce(0.0, Double::sum);
    }

    // Create comma-separated employee names
    public String namesWithReduce(List<Employee> employees) {
        return employees.stream()
            .map(Employee::getName)
            .reduce((s1, s2) -> s1 + ", " + s2)
            .orElse("");
    }

    // Combine all lists into one
    public List<Integer> combineLists(List<List<Integer>> lists) {
        return lists.stream()
            .reduce(new ArrayList<>(), (acc, list) -> {
                acc.addAll(list);
                return acc;
            });
    }
}
```

### Problem 19: Collect to Custom Collection

```java
public class CustomCollectors {

    // Collect to LinkedList
    public LinkedList<Employee> toLinkedList(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.toCollection(LinkedList::new));
    }

    // Collect to TreeSet (sorted)
    public TreeSet<String> toSortedSet(List<Employee> employees) {
        return employees.stream()
            .map(Employee::getName)
            .collect(Collectors.toCollection(TreeSet::new));
    }

    // Collect to LinkedHashMap (maintain insertion order)
    public Map<Long, Employee> toLinkedHashMap(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.toMap(
                Employee::getId,
                e -> e,
                (e1, e2) -> e1,
                LinkedHashMap::new
            ));
    }

    // Collect to ConcurrentMap (thread-safe)
    public Map<Long, Employee> toConcurrentMap(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.toConcurrentMap(
                Employee::getId,
                e -> e
            ));
    }
}
```

---

## Parallel Streams

### Problem 20: Parallel Processing

```java
public class ParallelStreamProblems {

    // Sum large list in parallel
    public long sumParallel(List<Integer> numbers) {
        return numbers.parallelStream()
            .mapToLong(Integer::longValue)
            .sum();
    }

    // Find in parallel (faster for large datasets)
    public Optional<Employee> findHighEarnerParallel(List<Employee> employees) {
        return employees.parallelStream()
            .filter(e -> e.getSalary() > 100000)
            .findAny();  // Use findAny() in parallel streams
    }

    // Process with custom thread pool
    public List<Employee> processWithPool(List<Employee> employees) throws Exception {
        ForkJoinPool customPool = new ForkJoinPool(4);
        try {
            return customPool.submit(() ->
                employees.parallelStream()
                    .filter(e -> e.getSalary() > 50000)
                    .collect(Collectors.toList())
            ).get();
        } finally {
            customPool.shutdown();
        }
    }

    // When NOT to use parallel streams
    public void parallelPitfalls() {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5);

        // Bad: Small dataset - overhead > benefit
        numbers.parallelStream().forEach(System.out::println);

        // Bad: Order matters - use sequential
        // numbers.parallelStream().forEachOrdered(System.out::println);

        // Bad: Shared mutable state - race condition
        // List<Integer> result = new ArrayList<>();
        // numbers.parallelStream().forEach(result::add);  // NOT THREAD-SAFE!

        // Good: Use collector for thread-safe collection
        List<Integer> result = numbers.parallelStream()
            .collect(Collectors.toList());
    }
}
```

---

## Real-World Scenarios

### Problem 21: Transaction Analysis

```java
public class TransactionAnalysis {

    // Get total credits and debits
    public Map<String, Double> creditDebitTotals(List<Transaction> transactions) {
        return transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getType,
                Collectors.summingDouble(Transaction::getAmount)
            ));
    }

    // Monthly transaction summary
    public Map<Month, Double> monthlySpending(List<Transaction> transactions) {
        return transactions.stream()
            .filter(t -> "DEBIT".equals(t.getType()))
            .collect(Collectors.groupingBy(
                t -> t.getDate().getMonth(),
                Collectors.summingDouble(Transaction::getAmount)
            ));
    }

    // Category-wise expense report
    public Map<String, DoubleSummaryStatistics> categoryStats(List<Transaction> transactions) {
        return transactions.stream()
            .filter(t -> "DEBIT".equals(t.getType()))
            .collect(Collectors.groupingBy(
                Transaction::getCategory,
                Collectors.summarizingDouble(Transaction::getAmount)
            ));
    }

    // Find highest transaction per category
    public Map<String, Optional<Transaction>> highestPerCategory(List<Transaction> transactions) {
        return transactions.stream()
            .collect(Collectors.groupingBy(
                Transaction::getCategory,
                Collectors.maxBy(Comparator.comparingDouble(Transaction::getAmount))
            ));
    }

    // Get transactions for last N days
    public List<Transaction> lastNDays(List<Transaction> transactions, int days) {
        LocalDate cutoff = LocalDate.now().minusDays(days);
        return transactions.stream()
            .filter(t -> t.getDate().isAfter(cutoff))
            .sorted(Comparator.comparing(Transaction::getDate).reversed())
            .collect(Collectors.toList());
    }
}
```

### Problem 22: Order Processing

```java
public class OrderProcessing {

    // Calculate order total
    public double orderTotal(Order order) {
        return order.getItems().stream()
            .mapToDouble(item -> item.getQuantity() * item.getPrice())
            .sum();
    }

    // Find customer with highest order value
    public Optional<String> topCustomer(List<Order> orders) {
        return orders.stream()
            .collect(Collectors.groupingBy(
                Order::getCustomerId,
                Collectors.summingDouble(this::orderTotal)
            ))
            .entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey);
    }

    // Get product sales quantity
    public Map<String, Integer> productQuantities(List<Order> orders) {
        return orders.stream()
            .flatMap(order -> order.getItems().stream())
            .collect(Collectors.groupingBy(
                OrderItem::getProductName,
                Collectors.summingInt(OrderItem::getQuantity)
            ));
    }

    // Find orders with more than N items
    public List<Order> ordersWithMoreThan(List<Order> orders, int count) {
        return orders.stream()
            .filter(order -> order.getItems().size() > count)
            .collect(Collectors.toList());
    }

    // Revenue by month
    public Map<YearMonth, Double> monthlyRevenue(List<Order> orders) {
        return orders.stream()
            .collect(Collectors.groupingBy(
                order -> YearMonth.from(order.getOrderDate()),
                Collectors.summingDouble(this::orderTotal)
            ));
    }
}
```

### Problem 23: Employee Department Analysis

```java
public class DepartmentAnalysis {

    // Find department with highest average salary
    public Optional<String> deptWithHighestAvgSalary(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.averagingDouble(Employee::getSalary)
            ))
            .entrySet().stream()
            .max(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey);
    }

    // Employee count per department sorted by count
    public List<Map.Entry<String, Long>> deptCountSorted(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()
            ))
            .entrySet().stream()
            .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
            .collect(Collectors.toList());
    }

    // Salary distribution (histogram)
    public Map<String, Long> salaryDistribution(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(e -> {
                double salary = e.getSalary();
                if (salary < 30000) return "0-30K";
                else if (salary < 50000) return "30K-50K";
                else if (salary < 70000) return "50K-70K";
                else if (salary < 100000) return "70K-100K";
                else return "100K+";
            }, Collectors.counting()));
    }

    // Create department summary report
    public Map<String, String> departmentSummary(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.collectingAndThen(
                    Collectors.toList(),
                    emps -> String.format(
                        "Count: %d, Avg Salary: %.2f, Youngest: %d, Oldest: %d",
                        emps.size(),
                        emps.stream().mapToDouble(Employee::getSalary).average().orElse(0),
                        emps.stream().mapToInt(Employee::getAge).min().orElse(0),
                        emps.stream().mapToInt(Employee::getAge).max().orElse(0)
                    )
                )
            ));
    }
}
```

### Problem 24: Combining Multiple Streams

```java
public class StreamCombining {

    // Merge two lists and remove duplicates
    public List<Integer> mergeLists(List<Integer> list1, List<Integer> list2) {
        return Stream.concat(list1.stream(), list2.stream())
            .distinct()
            .collect(Collectors.toList());
    }

    // Zip two lists (pair elements)
    public <T, U> List<Map.Entry<T, U>> zip(List<T> list1, List<U> list2) {
        return IntStream.range(0, Math.min(list1.size(), list2.size()))
            .mapToObj(i -> Map.entry(list1.get(i), list2.get(i)))
            .collect(Collectors.toList());
    }

    // Create pairs of employees by department
    public List<List<Employee>> pairsByDepartment(List<Employee> employees) {
        return employees.stream()
            .collect(Collectors.groupingBy(Employee::getDepartment))
            .values().stream()
            .filter(list -> list.size() >= 2)
            .map(list -> List.of(list.get(0), list.get(1)))
            .collect(Collectors.toList());
    }

    // Cross join two lists
    public <T, U> List<Map.Entry<T, U>> crossJoin(List<T> list1, List<U> list2) {
        return list1.stream()
            .flatMap(t -> list2.stream()
                .map(u -> Map.entry(t, u)))
            .collect(Collectors.toList());
    }
}
```

---

## Interview Questions & Answers

### Q1: What is the difference between intermediate and terminal operations?

| Aspect        | Intermediate                    | Terminal                             |
| ------------- | ------------------------------- | ------------------------------------ |
| **Returns**   | Stream                          | Non-stream (value, collection, void) |
| **Execution** | Lazy (not executed immediately) | Eager (triggers pipeline)            |
| **Chaining**  | Can be chained                  | Ends the pipeline                    |
| **Examples**  | filter, map, flatMap, sorted    | collect, forEach, reduce, count      |

```java
// Intermediate operations don't execute until terminal is called
Stream<Integer> stream = numbers.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);  // Won't print!
        return n > 5;
    });

// Terminal operation triggers execution
long count = stream.count();  // Now "Filtering: x" prints
```

---

### Q2: When should you use flatMap vs map?

```java
// map(): One-to-one transformation
List<String> names = employees.stream()
    .map(Employee::getName)      // Employee -> String
    .collect(Collectors.toList());

// flatMap(): One-to-many + flattening
List<String> allProducts = orders.stream()
    .flatMap(order -> order.getItems().stream())  // Flattens nested lists
    .map(OrderItem::getProductName)
    .collect(Collectors.toList());

// Rule: Use flatMap when you have nested collections
// Stream<List<T>> -> Stream<T>
```

---

### Q3: How do you handle null values in Streams?

```java
// Option 1: Filter out nulls
List<String> names = employees.stream()
    .filter(Objects::nonNull)
    .map(Employee::getName)
    .filter(Objects::nonNull)
    .collect(Collectors.toList());

// Option 2: Use Optional
List<String> names = employees.stream()
    .map(Employee::getName)
    .map(Optional::ofNullable)
    .flatMap(Optional::stream)  // Java 9+
    .collect(Collectors.toList());

// Option 3: Provide default value
List<String> names = employees.stream()
    .map(e -> e.getName() != null ? e.getName() : "Unknown")
    .collect(Collectors.toList());
```

---

### Q4: What are the differences between findFirst() and findAny()?

| Aspect            | findFirst()             | findAny()                    |
| ----------------- | ----------------------- | ---------------------------- |
| **Behavior**      | Returns first element   | Returns any matching element |
| **Deterministic** | Yes, always same result | No, may vary in parallel     |
| **Performance**   | Slower in parallel      | Faster in parallel           |
| **Use case**      | When order matters      | When any match is acceptable |

```java
// findFirst(): Deterministic, use when order matters
Optional<Employee> first = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .findFirst();

// findAny(): Faster in parallel, use for existence checks
Optional<Employee> any = employees.parallelStream()
    .filter(e -> e.getSalary() > 50000)
    .findAny();
```

---

### Q5: How do you convert Stream to Array?

```java
// To Object array
Object[] array = list.stream().toArray();

// To specific type array
String[] names = employees.stream()
    .map(Employee::getName)
    .toArray(String[]::new);

// To primitive array
int[] ages = employees.stream()
    .mapToInt(Employee::getAge)
    .toArray();

// From array to stream
Stream<String> stream = Arrays.stream(names);
IntStream intStream = Arrays.stream(ages);
```

---

### Q6: What is the difference between reduce() and collect()?

| Aspect          | reduce()                           | collect()                         |
| --------------- | ---------------------------------- | --------------------------------- |
| **Purpose**     | Combine elements into single value | Accumulate into mutable container |
| **Mutability**  | Works with immutable values        | Uses mutable accumulator          |
| **Use case**    | Sum, product, concatenation        | Lists, Maps, Sets                 |
| **Performance** | Good for simple aggregations       | Better for building collections   |

```java
// reduce(): Combine to single value
int sum = numbers.stream()
    .reduce(0, Integer::sum);

String concat = strings.stream()
    .reduce("", String::concat);

// collect(): Build mutable collection
List<String> list = stream.collect(Collectors.toList());

Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));
```

---

### Q7: When should you NOT use parallel streams?

```java
// 1. Small datasets - overhead exceeds benefit
List<Integer> small = List.of(1, 2, 3, 4, 5);
small.parallelStream().sum();  // Sequential is faster!

// 2. Order-dependent operations
// If you need forEachOrdered, don't use parallel
list.parallelStream().forEachOrdered(System.out::println);

// 3. Stateful operations
// peek(), limit(), skip() may give unexpected results

// 4. Shared mutable state - causes race conditions
List<Integer> result = new ArrayList<>();
numbers.parallelStream().forEach(result::add);  // WRONG!

// 5. I/O operations - already blocking
files.parallelStream().forEach(this::readFile);  // No benefit

// 6. LinkedList - poor splitting performance
linkedList.parallelStream();  // ArrayList is better

// SAFE: Use for CPU-intensive operations on large datasets
largeList.parallelStream()
    .filter(this::expensiveComputation)
    .collect(Collectors.toList());
```

---

### Q8: How do you debug streams?

```java
// Method 1: Use peek() for logging
List<Employee> result = employees.stream()
    .peek(e -> System.out.println("Before filter: " + e))
    .filter(e -> e.getSalary() > 50000)
    .peek(e -> System.out.println("After filter: " + e))
    .collect(Collectors.toList());

// Method 2: Break into smaller streams
Stream<Employee> filtered = employees.stream()
    .filter(e -> e.getSalary() > 50000);
List<Employee> list = filtered.collect(Collectors.toList());
System.out.println("Filtered count: " + list.size());

// Method 3: Use IDE debugger with "Trace Current Stream Chain"
// IntelliJ: Click on stream chain -> "Trace Current Stream Chain"

// Method 4: Custom peek with conditional logging
.peek(e -> {
    if (e.getSalary() > 100000) {
        log.debug("High earner: {}", e);
    }
})
```

---

### Q9: What is the difference between Stream.of() and Arrays.stream()?

```java
// Stream.of(): Varargs, works with any objects
Stream<String> s1 = Stream.of("a", "b", "c");
Stream<Integer> s2 = Stream.of(1, 2, 3);
Stream<int[]> s3 = Stream.of(new int[]{1, 2, 3});  // Stream of one array!

// Arrays.stream(): For arrays, handles primitives better
String[] arr = {"a", "b", "c"};
Stream<String> s4 = Arrays.stream(arr);
Stream<String> s5 = Arrays.stream(arr, 0, 2);  // Subset

int[] intArr = {1, 2, 3};
IntStream s6 = Arrays.stream(intArr);  // IntStream, not Stream<int[]>

// Key difference: Primitive arrays
int[] nums = {1, 2, 3};
Stream<int[]> wrong = Stream.of(nums);      // One element: the array itself!
IntStream correct = Arrays.stream(nums);     // Three elements: 1, 2, 3
```

---

### Q10: How do you create an infinite stream?

```java
// Stream.iterate() - with seed and function
Stream<Integer> evens = Stream.iterate(0, n -> n + 2);
evens.limit(10).forEach(System.out::println);  // 0, 2, 4, ... 18

// Stream.iterate() with predicate (Java 9+)
Stream<Integer> limited = Stream.iterate(0, n -> n < 20, n -> n + 2);

// Stream.generate() - with supplier
Stream<Double> randoms = Stream.generate(Math::random);
randoms.limit(5).forEach(System.out::println);

// Stream.generate() for repeated values
Stream<String> hellos = Stream.generate(() -> "Hello");

// Practical example: Fibonacci sequence
Stream.iterate(new long[]{0, 1}, p -> new long[]{p[1], p[0] + p[1]})
    .limit(20)
    .mapToLong(p -> p[0])
    .forEach(System.out::println);

// IMPORTANT: Always use limit() with infinite streams!
```

---

### Q11: Explain Collectors.collectingAndThen()

```java
// collectingAndThen: Perform final transformation after collection

// Get unmodifiable list
List<Employee> immutable = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));

// Count and return as String
String countStr = employees.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.counting(),
        count -> "Total: " + count
    ));

// Get random element from filtered list
Optional<Employee> randomHigh = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        list -> list.isEmpty() ? Optional.empty() :
            Optional.of(list.get(new Random().nextInt(list.size())))
    ));
```

---

### Q12: What is a custom Collector and when would you create one?

```java
// Custom collector for comma-separated values
Collector<Employee, StringJoiner, String> employeeJoiner =
    Collector.of(
        () -> new StringJoiner(", "),                    // Supplier
        (joiner, e) -> joiner.add(e.getName()),          // Accumulator
        (j1, j2) -> j1.merge(j2),                        // Combiner (for parallel)
        StringJoiner::toString                            // Finisher
    );

String names = employees.stream().collect(employeeJoiner);

// Custom collector to group by department with custom map
Collector<Employee, ?, TreeMap<String, List<Employee>>> customGrouping =
    Collector.of(
        TreeMap::new,
        (map, e) -> map.computeIfAbsent(
            e.getDepartment(), k -> new ArrayList<>()
        ).add(e),
        (m1, m2) -> {
            m2.forEach((k, v) -> m1.merge(k, v, (l1, l2) -> {
                l1.addAll(l2);
                return l1;
            }));
            return m1;
        }
    );

// When to create custom collector:
// 1. Complex accumulation logic not covered by built-in collectors
// 2. Need specific container type (TreeMap, ConcurrentHashMap)
// 3. Performance optimization for specific use case
// 4. Reusable collection logic
```

---

## Key Takeaways

| Problem Type   | Stream Operation                          | Example                                             |
| -------------- | ----------------------------------------- | --------------------------------------------------- |
| Filter data    | `filter()`                                | `filter(e -> e.getSalary() > 50000)`                |
| Transform      | `map()`                                   | `map(Employee::getName)`                            |
| Flatten nested | `flatMap()`                               | `flatMap(order -> order.getItems().stream())`       |
| Sort           | `sorted()`                                | `sorted(Comparator.comparing(Employee::getSalary))` |
| Limit/Skip     | `limit()`, `skip()`                       | `skip(5).limit(10)`                                 |
| Distinct       | `distinct()`                              | `distinct()`                                        |
| Group          | `Collectors.groupingBy()`                 | `groupingBy(Employee::getDepartment)`               |
| Partition      | `Collectors.partitioningBy()`             | `partitioningBy(e -> e.getSalary() > 50000)`        |
| Aggregate      | `count()`, `sum()`, `average()`           | `mapToDouble(Employee::getSalary).average()`        |
| Find           | `findFirst()`, `findAny()`                | `findFirst()`                                       |
| Match          | `anyMatch()`, `allMatch()`, `noneMatch()` | `anyMatch(e -> e.getAge() > 50)`                    |
| Reduce         | `reduce()`                                | `reduce(0, Integer::sum)`                           |
| Collect        | `collect()`                               | `collect(Collectors.toList())`                      |
| Join strings   | `Collectors.joining()`                    | `joining(", ", "[", "]")`                           |
| Statistics     | `summarizingDouble()`                     | `summarizingDouble(Employee::getSalary)`            |

---

## Practice Checklist

- [ ] Find duplicates in a list
- [ ] Find second highest salary
- [ ] Group employees by department
- [ ] Count frequency of elements
- [ ] Flatten nested collections
- [ ] Sort by multiple criteria
- [ ] Calculate aggregates (sum, avg, min, max)
- [ ] Partition data by condition
- [ ] Join strings with delimiter
- [ ] Transform Map to different structure
- [ ] Find first non-repeating character
- [ ] Process transactions by category
- [ ] Handle null values safely
- [ ] Create pagination with skip/limit
- [ ] Combine multiple streams
