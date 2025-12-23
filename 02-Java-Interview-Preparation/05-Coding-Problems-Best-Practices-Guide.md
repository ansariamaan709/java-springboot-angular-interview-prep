# Java Coding Problems & Best Practices - Complete Interview Guide

## Table of Contents

1. [String Manipulation Problems](#string-manipulation-problems)
2. [Array and Matrix Problems](#array-and-matrix-problems)
3. [LinkedList Problems](#linkedlist-problems)
4. [Stack and Queue Problems](#stack-and-queue-problems)
5. [Tree and Graph Problems](#tree-and-graph-problems)
6. [Dynamic Programming Problems](#dynamic-programming-problems)
7. [Java Best Practices](#java-best-practices)
8. [Code Review Checklist](#code-review-checklist)

---

## String Manipulation Problems

### Problem 1: Reverse a String

```java
public class StringReversal {

    // Method 1: Using StringBuilder (Most efficient)
    public String reverseWithStringBuilder(String str) {
        if (str == null) return null;
        return new StringBuilder(str).reverse().toString();
    }

    // Method 2: Using char array (No extra object)
    public String reverseWithCharArray(String str) {
        if (str == null) return null;
        char[] chars = str.toCharArray();
        int left = 0, right = chars.length - 1;
        while (left < right) {
            char temp = chars[left];
            chars[left] = chars[right];
            chars[right] = temp;
            left++;
            right--;
        }
        return new String(chars);
    }

    // Method 3: Recursive (Not recommended - stack overflow for large strings)
    public String reverseRecursive(String str) {
        if (str == null || str.length() <= 1) return str;
        return reverseRecursive(str.substring(1)) + str.charAt(0);
    }

    // Method 4: Using Streams (Java 8+)
    public String reverseWithStreams(String str) {
        if (str == null) return null;
        return str.chars()
            .mapToObj(c -> String.valueOf((char) c))
            .reduce("", (s1, s2) -> s2 + s1);
    }
}
```

### Problem 2: Check if String is Palindrome

```java
public class PalindromeChecker {

    // Basic palindrome check
    public boolean isPalindrome(String str) {
        if (str == null) return false;
        str = str.toLowerCase().replaceAll("[^a-z0-9]", "");
        int left = 0, right = str.length() - 1;
        while (left < right) {
            if (str.charAt(left++) != str.charAt(right--)) {
                return false;
            }
        }
        return true;
    }

    // Using StringBuilder
    public boolean isPalindromeWithBuilder(String str) {
        if (str == null) return false;
        String cleaned = str.toLowerCase().replaceAll("[^a-z0-9]", "");
        return cleaned.equals(new StringBuilder(cleaned).reverse().toString());
    }

    // Check if string can be palindrome by removing at most one char
    public boolean validPalindromeII(String s) {
        int left = 0, right = s.length() - 1;
        while (left < right) {
            if (s.charAt(left) != s.charAt(right)) {
                return isPalindromeRange(s, left + 1, right) ||
                       isPalindromeRange(s, left, right - 1);
            }
            left++;
            right--;
        }
        return true;
    }

    private boolean isPalindromeRange(String s, int left, int right) {
        while (left < right) {
            if (s.charAt(left++) != s.charAt(right--)) return false;
        }
        return true;
    }
}
```

### Problem 3: Find All Anagrams in a String

```java
public class AnagramFinder {

    // Check if two strings are anagrams
    public boolean areAnagrams(String s1, String s2) {
        if (s1 == null || s2 == null || s1.length() != s2.length()) {
            return false;
        }

        int[] charCount = new int[26];
        for (int i = 0; i < s1.length(); i++) {
            charCount[s1.charAt(i) - 'a']++;
            charCount[s2.charAt(i) - 'a']--;
        }

        for (int count : charCount) {
            if (count != 0) return false;
        }
        return true;
    }

    // Find all starting indices of anagrams of pattern in text
    // Sliding Window approach - O(n)
    public List<Integer> findAllAnagrams(String text, String pattern) {
        List<Integer> result = new ArrayList<>();
        if (text == null || pattern == null || text.length() < pattern.length()) {
            return result;
        }

        int[] patternCount = new int[26];
        int[] windowCount = new int[26];

        // Count pattern characters
        for (char c : pattern.toCharArray()) {
            patternCount[c - 'a']++;
        }

        int windowSize = pattern.length();
        for (int i = 0; i < text.length(); i++) {
            // Add current character to window
            windowCount[text.charAt(i) - 'a']++;

            // Remove leftmost character if window exceeds size
            if (i >= windowSize) {
                windowCount[text.charAt(i - windowSize) - 'a']--;
            }

            // Check if current window is anagram
            if (i >= windowSize - 1 && Arrays.equals(patternCount, windowCount)) {
                result.add(i - windowSize + 1);
            }
        }

        return result;
    }

    // Group anagrams together
    public List<List<String>> groupAnagrams(String[] strs) {
        if (strs == null || strs.length == 0) {
            return new ArrayList<>();
        }

        Map<String, List<String>> map = new HashMap<>();
        for (String str : strs) {
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);

            map.computeIfAbsent(key, k -> new ArrayList<>()).add(str);
        }

        return new ArrayList<>(map.values());
    }
}
```

### Problem 4: Longest Substring Without Repeating Characters

```java
public class LongestSubstring {

    // Sliding Window with HashMap - O(n)
    public int lengthOfLongestSubstring(String s) {
        if (s == null || s.isEmpty()) return 0;

        Map<Character, Integer> lastSeen = new HashMap<>();
        int maxLength = 0;
        int windowStart = 0;

        for (int windowEnd = 0; windowEnd < s.length(); windowEnd++) {
            char currentChar = s.charAt(windowEnd);

            // If character was seen and is in current window
            if (lastSeen.containsKey(currentChar) &&
                lastSeen.get(currentChar) >= windowStart) {
                windowStart = lastSeen.get(currentChar) + 1;
            }

            lastSeen.put(currentChar, windowEnd);
            maxLength = Math.max(maxLength, windowEnd - windowStart + 1);
        }

        return maxLength;
    }

    // Optimized with array (for ASCII characters)
    public int lengthOfLongestSubstringOptimized(String s) {
        if (s == null || s.isEmpty()) return 0;

        int[] lastIndex = new int[128];  // ASCII
        Arrays.fill(lastIndex, -1);

        int maxLength = 0;
        int windowStart = 0;

        for (int windowEnd = 0; windowEnd < s.length(); windowEnd++) {
            char c = s.charAt(windowEnd);

            if (lastIndex[c] >= windowStart) {
                windowStart = lastIndex[c] + 1;
            }

            lastIndex[c] = windowEnd;
            maxLength = Math.max(maxLength, windowEnd - windowStart + 1);
        }

        return maxLength;
    }
}
```

---

## Array and Matrix Problems

### Problem 5: Two Sum

```java
public class TwoSum {

    // Using HashMap - O(n) time, O(n) space
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            if (map.containsKey(complement)) {
                return new int[] { map.get(complement), i };
            }
            map.put(nums[i], i);
        }

        throw new IllegalArgumentException("No solution found");
    }

    // For sorted array - Two Pointers - O(n) time, O(1) space
    public int[] twoSumSorted(int[] nums, int target) {
        int left = 0, right = nums.length - 1;

        while (left < right) {
            int sum = nums[left] + nums[right];
            if (sum == target) {
                return new int[] { left, right };
            } else if (sum < target) {
                left++;
            } else {
                right--;
            }
        }

        throw new IllegalArgumentException("No solution found");
    }

    // Three Sum - Find all unique triplets that sum to zero
    public List<List<Integer>> threeSum(int[] nums) {
        List<List<Integer>> result = new ArrayList<>();
        Arrays.sort(nums);

        for (int i = 0; i < nums.length - 2; i++) {
            // Skip duplicates for i
            if (i > 0 && nums[i] == nums[i - 1]) continue;

            int left = i + 1, right = nums.length - 1;
            int target = -nums[i];

            while (left < right) {
                int sum = nums[left] + nums[right];
                if (sum == target) {
                    result.add(Arrays.asList(nums[i], nums[left], nums[right]));

                    // Skip duplicates
                    while (left < right && nums[left] == nums[left + 1]) left++;
                    while (left < right && nums[right] == nums[right - 1]) right--;

                    left++;
                    right--;
                } else if (sum < target) {
                    left++;
                } else {
                    right--;
                }
            }
        }

        return result;
    }
}
```

### Problem 6: Maximum Subarray (Kadane's Algorithm)

```java
public class MaximumSubarray {

    // Kadane's Algorithm - O(n)
    public int maxSubArray(int[] nums) {
        if (nums == null || nums.length == 0) {
            throw new IllegalArgumentException("Empty array");
        }

        int maxSoFar = nums[0];
        int maxEndingHere = nums[0];

        for (int i = 1; i < nums.length; i++) {
            // Either extend current subarray or start new
            maxEndingHere = Math.max(nums[i], maxEndingHere + nums[i]);
            maxSoFar = Math.max(maxSoFar, maxEndingHere);
        }

        return maxSoFar;
    }

    // Return the actual subarray
    public int[] maxSubArrayWithIndices(int[] nums) {
        int maxSoFar = nums[0];
        int maxEndingHere = nums[0];
        int start = 0, end = 0, tempStart = 0;

        for (int i = 1; i < nums.length; i++) {
            if (nums[i] > maxEndingHere + nums[i]) {
                maxEndingHere = nums[i];
                tempStart = i;
            } else {
                maxEndingHere = maxEndingHere + nums[i];
            }

            if (maxEndingHere > maxSoFar) {
                maxSoFar = maxEndingHere;
                start = tempStart;
                end = i;
            }
        }

        return Arrays.copyOfRange(nums, start, end + 1);
    }

    // Maximum Circular Subarray Sum
    public int maxSubarraySumCircular(int[] nums) {
        int maxKadane = maxSubArray(nums);

        // If all negative, return max element
        if (maxKadane < 0) return maxKadane;

        // Otherwise, compare with wraparound case
        int totalSum = 0;
        for (int i = 0; i < nums.length; i++) {
            totalSum += nums[i];
            nums[i] = -nums[i];  // Negate for finding min subarray
        }

        int minKadane = maxSubArray(nums);  // This gives -minSubarray
        int wraparoundSum = totalSum + minKadane;  // total - minSubarray

        return Math.max(maxKadane, wraparoundSum);
    }
}
```

### Problem 7: Rotate Matrix 90 Degrees

```java
public class MatrixRotation {

    // Rotate matrix 90 degrees clockwise (in-place)
    // Transpose + Reverse each row
    public void rotate(int[][] matrix) {
        int n = matrix.length;

        // Transpose
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }

        // Reverse each row
        for (int i = 0; i < n; i++) {
            int left = 0, right = n - 1;
            while (left < right) {
                int temp = matrix[i][left];
                matrix[i][left] = matrix[i][right];
                matrix[i][right] = temp;
                left++;
                right--;
            }
        }
    }

    // Rotate 90 degrees counter-clockwise
    // Transpose + Reverse each column
    public void rotateCounterClockwise(int[][] matrix) {
        int n = matrix.length;

        // Transpose
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int temp = matrix[i][j];
                matrix[i][j] = matrix[j][i];
                matrix[j][i] = temp;
            }
        }

        // Reverse each column
        for (int j = 0; j < n; j++) {
            int top = 0, bottom = n - 1;
            while (top < bottom) {
                int temp = matrix[top][j];
                matrix[top][j] = matrix[bottom][j];
                matrix[bottom][j] = temp;
                top++;
                bottom--;
            }
        }
    }

    // Spiral Order Traversal
    public List<Integer> spiralOrder(int[][] matrix) {
        List<Integer> result = new ArrayList<>();
        if (matrix == null || matrix.length == 0) return result;

        int top = 0, bottom = matrix.length - 1;
        int left = 0, right = matrix[0].length - 1;

        while (top <= bottom && left <= right) {
            // Traverse right
            for (int i = left; i <= right; i++) {
                result.add(matrix[top][i]);
            }
            top++;

            // Traverse down
            for (int i = top; i <= bottom; i++) {
                result.add(matrix[i][right]);
            }
            right--;

            // Traverse left
            if (top <= bottom) {
                for (int i = right; i >= left; i--) {
                    result.add(matrix[bottom][i]);
                }
                bottom--;
            }

            // Traverse up
            if (left <= right) {
                for (int i = bottom; i >= top; i--) {
                    result.add(matrix[i][left]);
                }
                left++;
            }
        }

        return result;
    }
}
```

### Problem 8: Merge Intervals

```java
public class MergeIntervals {

    public int[][] merge(int[][] intervals) {
        if (intervals == null || intervals.length <= 1) {
            return intervals;
        }

        // Sort by start time
        Arrays.sort(intervals, (a, b) -> Integer.compare(a[0], b[0]));

        List<int[]> merged = new ArrayList<>();
        int[] current = intervals[0];

        for (int i = 1; i < intervals.length; i++) {
            if (intervals[i][0] <= current[1]) {
                // Overlapping, merge
                current[1] = Math.max(current[1], intervals[i][1]);
            } else {
                // Non-overlapping, add current and move to next
                merged.add(current);
                current = intervals[i];
            }
        }
        merged.add(current);  // Add last interval

        return merged.toArray(new int[merged.size()][]);
    }

    // Insert interval and merge
    public int[][] insert(int[][] intervals, int[] newInterval) {
        List<int[]> result = new ArrayList<>();
        int i = 0;
        int n = intervals.length;

        // Add all intervals ending before newInterval starts
        while (i < n && intervals[i][1] < newInterval[0]) {
            result.add(intervals[i++]);
        }

        // Merge overlapping intervals
        while (i < n && intervals[i][0] <= newInterval[1]) {
            newInterval[0] = Math.min(newInterval[0], intervals[i][0]);
            newInterval[1] = Math.max(newInterval[1], intervals[i][1]);
            i++;
        }
        result.add(newInterval);

        // Add remaining intervals
        while (i < n) {
            result.add(intervals[i++]);
        }

        return result.toArray(new int[result.size()][]);
    }
}
```

---

## LinkedList Problems

### Problem 9: Reverse LinkedList

```java
public class LinkedListProblems {

    static class ListNode {
        int val;
        ListNode next;
        ListNode(int val) { this.val = val; }
    }

    // Iterative reversal - O(n) time, O(1) space
    public ListNode reverseIterative(ListNode head) {
        ListNode prev = null;
        ListNode current = head;

        while (current != null) {
            ListNode nextTemp = current.next;
            current.next = prev;
            prev = current;
            current = nextTemp;
        }

        return prev;
    }

    // Recursive reversal - O(n) time, O(n) space (call stack)
    public ListNode reverseRecursive(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode newHead = reverseRecursive(head.next);
        head.next.next = head;
        head.next = null;

        return newHead;
    }

    // Reverse nodes in groups of k
    public ListNode reverseKGroup(ListNode head, int k) {
        if (head == null || k == 1) return head;

        // Count nodes
        int count = 0;
        ListNode curr = head;
        while (curr != null) {
            count++;
            curr = curr.next;
        }

        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode prevGroupEnd = dummy;

        while (count >= k) {
            ListNode groupStart = prevGroupEnd.next;
            ListNode prev = null;
            curr = groupStart;

            // Reverse k nodes
            for (int i = 0; i < k; i++) {
                ListNode next = curr.next;
                curr.next = prev;
                prev = curr;
                curr = next;
            }

            // Connect with previous group
            prevGroupEnd.next = prev;
            groupStart.next = curr;
            prevGroupEnd = groupStart;

            count -= k;
        }

        return dummy.next;
    }
}
```

### Problem 10: Detect Cycle in LinkedList

```java
public class LinkedListCycle {

    // Floyd's Cycle Detection (Tortoise and Hare)
    public boolean hasCycle(ListNode head) {
        if (head == null || head.next == null) return false;

        ListNode slow = head;
        ListNode fast = head;

        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) return true;
        }

        return false;
    }

    // Find the start of cycle
    public ListNode detectCycle(ListNode head) {
        if (head == null || head.next == null) return null;

        ListNode slow = head;
        ListNode fast = head;

        // Detect cycle
        while (fast != null && fast.next != null) {
            slow = slow.next;
            fast = fast.next.next;

            if (slow == fast) {
                // Found cycle, now find start
                ListNode start = head;
                while (start != slow) {
                    start = start.next;
                    slow = slow.next;
                }
                return start;
            }
        }

        return null;
    }

    // Find length of cycle
    public int cycleLength(ListNode head) {
        ListNode meetingPoint = detectCycle(head);
        if (meetingPoint == null) return 0;

        int length = 1;
        ListNode curr = meetingPoint.next;
        while (curr != meetingPoint) {
            length++;
            curr = curr.next;
        }

        return length;
    }
}
```

### Problem 11: Merge Two Sorted Lists

```java
public class MergeSortedLists {

    // Iterative approach
    public ListNode mergeTwoLists(ListNode l1, ListNode l2) {
        ListNode dummy = new ListNode(0);
        ListNode current = dummy;

        while (l1 != null && l2 != null) {
            if (l1.val <= l2.val) {
                current.next = l1;
                l1 = l1.next;
            } else {
                current.next = l2;
                l2 = l2.next;
            }
            current = current.next;
        }

        current.next = (l1 != null) ? l1 : l2;

        return dummy.next;
    }

    // Merge K sorted lists using Min Heap - O(N log k)
    public ListNode mergeKLists(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;

        PriorityQueue<ListNode> minHeap = new PriorityQueue<>(
            Comparator.comparingInt(a -> a.val)
        );

        // Add first node of each list
        for (ListNode node : lists) {
            if (node != null) {
                minHeap.offer(node);
            }
        }

        ListNode dummy = new ListNode(0);
        ListNode current = dummy;

        while (!minHeap.isEmpty()) {
            ListNode smallest = minHeap.poll();
            current.next = smallest;
            current = current.next;

            if (smallest.next != null) {
                minHeap.offer(smallest.next);
            }
        }

        return dummy.next;
    }

    // Merge K lists using Divide and Conquer - O(N log k)
    public ListNode mergeKListsDivideConquer(ListNode[] lists) {
        if (lists == null || lists.length == 0) return null;
        return mergeKListsHelper(lists, 0, lists.length - 1);
    }

    private ListNode mergeKListsHelper(ListNode[] lists, int start, int end) {
        if (start == end) return lists[start];
        if (start > end) return null;

        int mid = start + (end - start) / 2;
        ListNode left = mergeKListsHelper(lists, start, mid);
        ListNode right = mergeKListsHelper(lists, mid + 1, end);

        return mergeTwoLists(left, right);
    }
}
```

---

## Stack and Queue Problems

### Problem 12: Valid Parentheses

```java
public class ValidParentheses {

    public boolean isValid(String s) {
        Stack<Character> stack = new Stack<>();
        Map<Character, Character> mapping = Map.of(
            ')', '(',
            '}', '{',
            ']', '['
        );

        for (char c : s.toCharArray()) {
            if (mapping.containsKey(c)) {
                // Closing bracket
                if (stack.isEmpty() || stack.pop() != mapping.get(c)) {
                    return false;
                }
            } else {
                // Opening bracket
                stack.push(c);
            }
        }

        return stack.isEmpty();
    }

    // Generate all valid parentheses combinations
    public List<String> generateParenthesis(int n) {
        List<String> result = new ArrayList<>();
        backtrack(result, new StringBuilder(), 0, 0, n);
        return result;
    }

    private void backtrack(List<String> result, StringBuilder current,
                          int open, int close, int max) {
        if (current.length() == max * 2) {
            result.add(current.toString());
            return;
        }

        if (open < max) {
            current.append('(');
            backtrack(result, current, open + 1, close, max);
            current.deleteCharAt(current.length() - 1);
        }

        if (close < open) {
            current.append(')');
            backtrack(result, current, open, close + 1, max);
            current.deleteCharAt(current.length() - 1);
        }
    }

    // Longest valid parentheses substring
    public int longestValidParentheses(String s) {
        Stack<Integer> stack = new Stack<>();
        stack.push(-1);  // Base for valid substring
        int maxLen = 0;

        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                stack.push(i);
            } else {
                stack.pop();
                if (stack.isEmpty()) {
                    stack.push(i);  // New base
                } else {
                    maxLen = Math.max(maxLen, i - stack.peek());
                }
            }
        }

        return maxLen;
    }
}
```

### Problem 13: Implement Queue using Stacks

```java
public class QueueUsingStacks {

    private Stack<Integer> inputStack;
    private Stack<Integer> outputStack;

    public QueueUsingStacks() {
        inputStack = new Stack<>();
        outputStack = new Stack<>();
    }

    // Amortized O(1)
    public void push(int x) {
        inputStack.push(x);
    }

    // Amortized O(1)
    public int pop() {
        ensureOutputStack();
        return outputStack.pop();
    }

    // Amortized O(1)
    public int peek() {
        ensureOutputStack();
        return outputStack.peek();
    }

    public boolean empty() {
        return inputStack.isEmpty() && outputStack.isEmpty();
    }

    private void ensureOutputStack() {
        if (outputStack.isEmpty()) {
            while (!inputStack.isEmpty()) {
                outputStack.push(inputStack.pop());
            }
        }
    }
}

// Implement Stack using Queues
public class StackUsingQueues {

    private Queue<Integer> queue;

    public StackUsingQueues() {
        queue = new LinkedList<>();
    }

    // O(n) push, O(1) pop
    public void push(int x) {
        queue.offer(x);
        // Rotate to make x the front
        for (int i = 0; i < queue.size() - 1; i++) {
            queue.offer(queue.poll());
        }
    }

    public int pop() {
        return queue.poll();
    }

    public int top() {
        return queue.peek();
    }

    public boolean empty() {
        return queue.isEmpty();
    }
}
```

### Problem 14: Min Stack

```java
public class MinStack {

    private Stack<Integer> stack;
    private Stack<Integer> minStack;

    public MinStack() {
        stack = new Stack<>();
        minStack = new Stack<>();
    }

    public void push(int val) {
        stack.push(val);
        if (minStack.isEmpty() || val <= minStack.peek()) {
            minStack.push(val);
        }
    }

    public void pop() {
        int popped = stack.pop();
        if (popped == minStack.peek()) {
            minStack.pop();
        }
    }

    public int top() {
        return stack.peek();
    }

    public int getMin() {
        return minStack.peek();
    }
}

// Space-optimized MinStack using single stack
public class MinStackOptimized {

    private Stack<Long> stack;
    private long min;

    public MinStackOptimized() {
        stack = new Stack<>();
    }

    public void push(int val) {
        if (stack.isEmpty()) {
            stack.push(0L);
            min = val;
        } else {
            stack.push(val - min);  // Store difference
            if (val < min) {
                min = val;
            }
        }
    }

    public void pop() {
        long top = stack.pop();
        if (top < 0) {
            min = min - top;  // Restore previous min
        }
    }

    public int top() {
        long top = stack.peek();
        if (top < 0) {
            return (int) min;
        }
        return (int) (top + min);
    }

    public int getMin() {
        return (int) min;
    }
}
```

---

## Tree and Graph Problems

### Problem 15: Binary Tree Traversals

```java
public class TreeTraversals {

    static class TreeNode {
        int val;
        TreeNode left, right;
        TreeNode(int val) { this.val = val; }
    }

    // Inorder: Left, Root, Right
    public List<Integer> inorderIterative(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode curr = root;

        while (curr != null || !stack.isEmpty()) {
            while (curr != null) {
                stack.push(curr);
                curr = curr.left;
            }
            curr = stack.pop();
            result.add(curr.val);
            curr = curr.right;
        }

        return result;
    }

    // Preorder: Root, Left, Right
    public List<Integer> preorderIterative(TreeNode root) {
        List<Integer> result = new ArrayList<>();
        if (root == null) return result;

        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);

        while (!stack.isEmpty()) {
            TreeNode node = stack.pop();
            result.add(node.val);

            if (node.right != null) stack.push(node.right);
            if (node.left != null) stack.push(node.left);
        }

        return result;
    }

    // Postorder: Left, Right, Root
    public List<Integer> postorderIterative(TreeNode root) {
        LinkedList<Integer> result = new LinkedList<>();
        if (root == null) return result;

        Stack<TreeNode> stack = new Stack<>();
        stack.push(root);

        while (!stack.isEmpty()) {
            TreeNode node = stack.pop();
            result.addFirst(node.val);  // Add to front

            if (node.left != null) stack.push(node.left);
            if (node.right != null) stack.push(node.right);
        }

        return result;
    }

    // Level Order (BFS)
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> result = new ArrayList<>();
        if (root == null) return result;

        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        while (!queue.isEmpty()) {
            int levelSize = queue.size();
            List<Integer> level = new ArrayList<>();

            for (int i = 0; i < levelSize; i++) {
                TreeNode node = queue.poll();
                level.add(node.val);

                if (node.left != null) queue.offer(node.left);
                if (node.right != null) queue.offer(node.right);
            }

            result.add(level);
        }

        return result;
    }
}
```

### Problem 16: Lowest Common Ancestor

```java
public class LowestCommonAncestor {

    // LCA in Binary Tree
    public TreeNode lowestCommonAncestor(TreeNode root, TreeNode p, TreeNode q) {
        if (root == null || root == p || root == q) {
            return root;
        }

        TreeNode left = lowestCommonAncestor(root.left, p, q);
        TreeNode right = lowestCommonAncestor(root.right, p, q);

        if (left != null && right != null) {
            return root;  // p and q are in different subtrees
        }

        return (left != null) ? left : right;
    }

    // LCA in Binary Search Tree (optimized)
    public TreeNode lowestCommonAncestorBST(TreeNode root, TreeNode p, TreeNode q) {
        while (root != null) {
            if (p.val < root.val && q.val < root.val) {
                root = root.left;
            } else if (p.val > root.val && q.val > root.val) {
                root = root.right;
            } else {
                return root;
            }
        }
        return null;
    }

    // Check if tree is valid BST
    public boolean isValidBST(TreeNode root) {
        return isValidBST(root, Long.MIN_VALUE, Long.MAX_VALUE);
    }

    private boolean isValidBST(TreeNode node, long min, long max) {
        if (node == null) return true;
        if (node.val <= min || node.val >= max) return false;

        return isValidBST(node.left, min, node.val) &&
               isValidBST(node.right, node.val, max);
    }
}
```

### Problem 17: Graph BFS/DFS

```java
public class GraphTraversal {

    // BFS - Shortest path in unweighted graph
    public int shortestPath(Map<Integer, List<Integer>> graph, int start, int end) {
        if (start == end) return 0;

        Set<Integer> visited = new HashSet<>();
        Queue<int[]> queue = new LinkedList<>();  // {node, distance}
        queue.offer(new int[]{start, 0});
        visited.add(start);

        while (!queue.isEmpty()) {
            int[] current = queue.poll();
            int node = current[0];
            int distance = current[1];

            for (int neighbor : graph.getOrDefault(node, Collections.emptyList())) {
                if (neighbor == end) {
                    return distance + 1;
                }

                if (!visited.contains(neighbor)) {
                    visited.add(neighbor);
                    queue.offer(new int[]{neighbor, distance + 1});
                }
            }
        }

        return -1;  // No path found
    }

    // DFS - Detect cycle in directed graph
    public boolean hasCycleDirected(Map<Integer, List<Integer>> graph, int numNodes) {
        int[] state = new int[numNodes];  // 0: unvisited, 1: visiting, 2: visited

        for (int i = 0; i < numNodes; i++) {
            if (state[i] == 0 && hasCycleDFS(graph, i, state)) {
                return true;
            }
        }

        return false;
    }

    private boolean hasCycleDFS(Map<Integer, List<Integer>> graph, int node, int[] state) {
        state[node] = 1;  // Visiting

        for (int neighbor : graph.getOrDefault(node, Collections.emptyList())) {
            if (state[neighbor] == 1) {
                return true;  // Back edge found
            }
            if (state[neighbor] == 0 && hasCycleDFS(graph, neighbor, state)) {
                return true;
            }
        }

        state[node] = 2;  // Visited
        return false;
    }

    // Topological Sort using DFS
    public List<Integer> topologicalSort(Map<Integer, List<Integer>> graph, int numNodes) {
        List<Integer> result = new ArrayList<>();
        boolean[] visited = new boolean[numNodes];
        Stack<Integer> stack = new Stack<>();

        for (int i = 0; i < numNodes; i++) {
            if (!visited[i]) {
                topologicalSortDFS(graph, i, visited, stack);
            }
        }

        while (!stack.isEmpty()) {
            result.add(stack.pop());
        }

        return result;
    }

    private void topologicalSortDFS(Map<Integer, List<Integer>> graph, int node,
                                    boolean[] visited, Stack<Integer> stack) {
        visited[node] = true;

        for (int neighbor : graph.getOrDefault(node, Collections.emptyList())) {
            if (!visited[neighbor]) {
                topologicalSortDFS(graph, neighbor, visited, stack);
            }
        }

        stack.push(node);
    }
}
```

---

## Dynamic Programming Problems

### Problem 18: Fibonacci Variations

```java
public class FibonacciProblems {

    // Climbing Stairs (same as Fibonacci)
    public int climbStairs(int n) {
        if (n <= 2) return n;

        int prev2 = 1, prev1 = 2;
        for (int i = 3; i <= n; i++) {
            int current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }

    // Minimum cost climbing stairs
    public int minCostClimbingStairs(int[] cost) {
        int n = cost.length;
        int prev2 = cost[0], prev1 = cost[1];

        for (int i = 2; i < n; i++) {
            int current = cost[i] + Math.min(prev1, prev2);
            prev2 = prev1;
            prev1 = current;
        }

        return Math.min(prev1, prev2);
    }

    // House Robber (can't rob adjacent houses)
    public int rob(int[] nums) {
        if (nums.length == 1) return nums[0];

        int prev2 = nums[0];
        int prev1 = Math.max(nums[0], nums[1]);

        for (int i = 2; i < nums.length; i++) {
            int current = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }

    // House Robber II (houses in circle)
    public int robCircular(int[] nums) {
        if (nums.length == 1) return nums[0];

        return Math.max(
            robRange(nums, 0, nums.length - 2),  // Exclude last
            robRange(nums, 1, nums.length - 1)   // Exclude first
        );
    }

    private int robRange(int[] nums, int start, int end) {
        int prev2 = 0, prev1 = 0;
        for (int i = start; i <= end; i++) {
            int current = Math.max(prev1, prev2 + nums[i]);
            prev2 = prev1;
            prev1 = current;
        }
        return prev1;
    }
}
```

### Problem 19: Longest Common Subsequence

```java
public class LCSProblems {

    // Longest Common Subsequence
    public int longestCommonSubsequence(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[][] dp = new int[m + 1][n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1] + 1;
                } else {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
                }
            }
        }

        return dp[m][n];
    }

    // Space optimized - O(n) space
    public int longestCommonSubsequenceOptimized(String text1, String text2) {
        int m = text1.length(), n = text2.length();
        int[] prev = new int[n + 1];
        int[] curr = new int[n + 1];

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (text1.charAt(i - 1) == text2.charAt(j - 1)) {
                    curr[j] = prev[j - 1] + 1;
                } else {
                    curr[j] = Math.max(prev[j], curr[j - 1]);
                }
            }
            int[] temp = prev;
            prev = curr;
            curr = temp;
        }

        return prev[n];
    }

    // Longest Increasing Subsequence - O(n log n)
    public int lengthOfLIS(int[] nums) {
        List<Integer> tails = new ArrayList<>();

        for (int num : nums) {
            int pos = Collections.binarySearch(tails, num);
            if (pos < 0) {
                pos = -(pos + 1);
            }

            if (pos == tails.size()) {
                tails.add(num);
            } else {
                tails.set(pos, num);
            }
        }

        return tails.size();
    }
}
```

### Problem 20: Coin Change

```java
public class CoinChangeProblems {

    // Minimum coins to make amount
    public int coinChange(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, amount + 1);  // Max value
        dp[0] = 0;

        for (int i = 1; i <= amount; i++) {
            for (int coin : coins) {
                if (coin <= i) {
                    dp[i] = Math.min(dp[i], dp[i - coin] + 1);
                }
            }
        }

        return dp[amount] > amount ? -1 : dp[amount];
    }

    // Number of ways to make amount (combinations)
    public int coinChangeWays(int[] coins, int amount) {
        int[] dp = new int[amount + 1];
        dp[0] = 1;

        // Loop coins first to avoid counting permutations
        for (int coin : coins) {
            for (int i = coin; i <= amount; i++) {
                dp[i] += dp[i - coin];
            }
        }

        return dp[amount];
    }

    // 0/1 Knapsack
    public int knapsack(int[] weights, int[] values, int capacity) {
        int n = weights.length;
        int[][] dp = new int[n + 1][capacity + 1];

        for (int i = 1; i <= n; i++) {
            for (int w = 1; w <= capacity; w++) {
                if (weights[i - 1] <= w) {
                    dp[i][w] = Math.max(
                        dp[i - 1][w],  // Don't take
                        dp[i - 1][w - weights[i - 1]] + values[i - 1]  // Take
                    );
                } else {
                    dp[i][w] = dp[i - 1][w];
                }
            }
        }

        return dp[n][capacity];
    }
}
```

---

## Java Best Practices

### Naming Conventions

```java
// Classes: PascalCase
public class CustomerOrder { }
public class HttpRequestHandler { }

// Methods and variables: camelCase
public void calculateTotalPrice() { }
private int itemCount;

// Constants: UPPER_SNAKE_CASE
public static final int MAX_CONNECTIONS = 100;
public static final String API_BASE_URL = "https://api.example.com";

// Packages: lowercase
package com.company.project.module;

// Interfaces: Often adjectives or nouns
public interface Serializable { }
public interface UserRepository { }

// Boolean methods: use is, has, can, should
public boolean isEmpty() { }
public boolean hasPermission() { }
public boolean canExecute() { }
```

### Effective Java Practices

```java
// 1. Use static factory methods instead of constructors
public class User {
    private User(String name) { }

    public static User of(String name) {
        return new User(name);
    }

    public static User createGuest() {
        return new User("Guest");
    }
}

// 2. Use builders for complex object construction
@Builder
public class Order {
    private final Long id;
    private final String customerId;
    private final List<Item> items;
    private final BigDecimal total;
}

Order order = Order.builder()
    .id(1L)
    .customerId("C123")
    .items(items)
    .total(BigDecimal.valueOf(99.99))
    .build();

// 3. Prefer composition over inheritance
public class Stack<E> {
    private final List<E> list = new ArrayList<>();  // Composition

    public void push(E e) { list.add(e); }
    public E pop() { return list.remove(list.size() - 1); }
}

// 4. Design for immutability
public final class ImmutablePerson {
    private final String name;
    private final List<String> hobbies;

    public ImmutablePerson(String name, List<String> hobbies) {
        this.name = name;
        this.hobbies = List.copyOf(hobbies);  // Defensive copy
    }

    public String getName() { return name; }
    public List<String> getHobbies() { return hobbies; }  // Already immutable
}

// 5. Use try-with-resources
public String readFile(String path) throws IOException {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        return reader.lines().collect(Collectors.joining("\n"));
    }
}

// 6. Prefer Optional over null
public Optional<User> findById(Long id) {
    return Optional.ofNullable(userMap.get(id));
}

// Using Optional
findById(123L)
    .map(User::getName)
    .filter(name -> name.startsWith("A"))
    .ifPresent(System.out::println);

// 7. Use appropriate exception types
// Checked: recoverable conditions
public void readConfig() throws ConfigurationException { }

// Unchecked: programming errors
public void setAge(int age) {
    if (age < 0) throw new IllegalArgumentException("Age cannot be negative");
}
```

### Performance Best Practices

```java
// 1. Use primitive types instead of wrappers when possible
// Bad
Long sum = 0L;
for (long i = 0; i < 1_000_000; i++) {
    sum += i;  // Auto-boxing overhead
}

// Good
long sum = 0L;
for (long i = 0; i < 1_000_000; i++) {
    sum += i;
}

// 2. Use StringBuilder for string concatenation in loops
// Bad
String result = "";
for (String s : list) {
    result += s;  // Creates new String each time
}

// Good
StringBuilder sb = new StringBuilder();
for (String s : list) {
    sb.append(s);
}

// 3. Initialize collections with expected size
List<User> users = new ArrayList<>(expectedSize);
Map<String, User> userMap = new HashMap<>(expectedSize);

// 4. Use lazy initialization for expensive objects
private ExpensiveObject expensiveObject;

public ExpensiveObject getExpensiveObject() {
    if (expensiveObject == null) {
        expensiveObject = new ExpensiveObject();
    }
    return expensiveObject;
}

// 5. Avoid creating objects in loops
// Bad
for (int i = 0; i < 1000; i++) {
    Pattern pattern = Pattern.compile("\\d+");  // Compiled each iteration!
    pattern.matcher(input).matches();
}

// Good
private static final Pattern DIGIT_PATTERN = Pattern.compile("\\d+");
for (int i = 0; i < 1000; i++) {
    DIGIT_PATTERN.matcher(input).matches();
}

// 6. Use appropriate data structures
// For frequent lookups: HashSet/HashMap O(1)
// For sorted data: TreeSet/TreeMap O(log n)
// For frequent insertions/deletions: LinkedList O(1)
// For indexed access: ArrayList O(1)
```

---

## Code Review Checklist

### Functionality

- [ ] Code works as expected
- [ ] Edge cases handled
- [ ] Error handling is appropriate
- [ ] No null pointer exceptions possible

### Code Quality

- [ ] Single Responsibility Principle followed
- [ ] Methods are small and focused
- [ ] No code duplication (DRY)
- [ ] Meaningful variable/method names
- [ ] No magic numbers (use constants)

### Performance

- [ ] No unnecessary object creation
- [ ] Appropriate data structures used
- [ ] No N+1 queries
- [ ] Collections sized appropriately
- [ ] No memory leaks

### Security

- [ ] Input validated
- [ ] SQL injection prevented
- [ ] Sensitive data not logged
- [ ] Authentication/authorization checked

### Testing

- [ ] Unit tests written
- [ ] Edge cases tested
- [ ] Mocking used appropriately
- [ ] Test coverage adequate

### Documentation

- [ ] Complex logic commented
- [ ] Public API documented
- [ ] README updated if needed

---

## Key Takeaways

| Problem Type        | Key Technique          | Time Complexity |
| ------------------- | ---------------------- | --------------- |
| Two Sum             | HashMap                | O(n)            |
| Sliding Window      | Two pointers           | O(n)            |
| Tree Traversal      | Stack/Queue            | O(n)            |
| Graph BFS/DFS       | Visited set            | O(V + E)        |
| Dynamic Programming | Memoization/Tabulation | Varies          |
| Binary Search       | Divide and conquer     | O(log n)        |
| Sorting             | Quick/Merge sort       | O(n log n)      |
