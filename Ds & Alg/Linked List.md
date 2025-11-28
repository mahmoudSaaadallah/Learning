### What is a Linked List?

At its core, a linked list is a *linear* data structure, much like an array. However, unlike arrays, which store elements in *contiguous* memory locations, a linked list stores elements at *arbitrary* locations. The "link" comes from the fact that each element, often called a **node**, contains not only the data itself but also a reference (or "pointer") to the next node in the sequence.

	Think of it like a treasure hunt: each clue tells you where the next clue is, rather than having all the clues laid out in a row on a single map.

**Key Characteristics:**

*   **Dynamic Size**: Linked lists can grow or shrink in size during execution, unlike static arrays.
*   **Ease of Insertion/Deletion**: Adding or removing elements can be very efficient once the position is found, as it only requires updating a few pointers.
*   **No Random Access**: To access an element, you must traverse the list from the beginning (or end, in some cases) until you reach the desired element. This means accessing the $k$-th element takes $O(k)$ time, not $O(1)$ like arrays.
*   **Memory Overhead**: Each node requires extra memory to store the pointer(s) to the next (and previous) nodes.

### The Building Block: The Node

Before we build a linked list, we need its fundamental component: the `Node`. A node is an object that holds a piece of data and a reference to the next node.

Let's define a simple `Node` class in Python:

```python
class Node:
    def __init__(self, data):
        self.data = data  # The data the node holds
        self.next = None  # Reference to the next node in the list, initially None
```

This `Node` class is the foundation for both singly and doubly linked lists.

---

### 1. Singly Linked List

A **singly linked list** is the simplest form of a linked list. Each node in a singly linked list contains two fields:
1.  `data`: The actual value stored in the node.
2.  `next`: A pointer or reference to the next node in the sequence. The last node's `next` pointer is typically `None`, signifying the end of the list.

The list itself is managed by a `head` pointer, which points to the first node in the list. If the list is empty, `head` is `None`.

#### Python Implementation of a Singly Linked List

Let's build a `SinglyLinkedList` class with common operations.

```python
class SinglyLinkedList:
    def __init__(self):
        self.head = None  # The head of the list, initially empty
        self.length = 0   # Keep track of the list's length

    def __str__(self):
        """Provides a string representation of the list."""
        nodes = []
        current = self.head
        while current:
            nodes.append(str(current.data))
            current = current.next
        return " -> ".join(nodes)

    def append(self, data):
        """Adds a new node with the given data to the end of the list."""
        new_node = Node(data)
        if not self.head:
            self.head = new_node
        else:
            current = self.head
            while current.next:  # Traverse to the last node
                current = current.next
            current.next = new_node
        self.length += 1
        print(f"Appended {data}. List: {self}")

    def prepend(self, data):
        """Adds a new node with the given data to the beginning of the list."""
        new_node = Node(data)
        new_node.next = self.head  # New node points to the old head
        self.head = new_node      # New node becomes the head
        self.length += 1
        print(f"Prepended {data}. List: {self}")

    def insert_after(self, prev_data, data):
        """Inserts a new node with 'data' after the node containing 'prev_data'."""
        if not self.head:
            print("List is empty. Cannot insert.")
            return

        current = self.head
        while current and current.data != prev_data:
            current = current.next

        if current is None:
            print(f"Node with data {prev_data} not found.")
            return

        new_node = Node(data)
        new_node.next = current.next
        current.next = new_node
        self.length += 1
        print(f"Inserted {data} after {prev_data}. List: {self}")

    def delete_node(self, key):
        """Deletes the first node found with the given 'key' (data)."""
        current = self.head

        # Case 1: Head node itself holds the key to be deleted
        if current and current.data == key:
            self.head = current.next
            current = None
            self.length -= 1
            print(f"Deleted {key}. List: {self}")
            return

        prev = None
        while current and current.data != key:
            prev = current
            current = current.next

        # Case 2: Key not present in list
        if current is None:
            print(f"Node with data {key} not found.")
            return

        # Case 3: Key found, unlink the node
        prev.next = current.next
        current = None
        self.length -= 1
        print(f"Deleted {key}. List: {self}")

    def search(self, key):
        """Searches for a node with the given 'key'."""
        current = self.head
        position = 0
        while current:
            if current.data == key:
                print(f"Found {key} at position {position}.")
                return True
            current = current.next
            position += 1
        print(f"{key} not found in the list.")
        return False

    def get_length(self):
        """Returns the current length of the list."""
        print(f"List length: {self.length}")
        return self.length

# --- Singly Linked List Examples ---
print("\n--- Singly Linked List Operations ---")
sll = SinglyLinkedList()
sll.append(10)  # List: 10
sll.append(20)  # List: 10 -> 20
sll.prepend(5)  # List: 5 -> 10 -> 20
sll.insert_after(10, 15) # List: 5 -> 10 -> 15 -> 20
sll.delete_node(5) # List: 10 -> 15 -> 20
sll.delete_node(25) # 25 not found
sll.search(15) # Found 15
sll.search(30) # 30 not found
sll.get_length() # Length: 3
sll.delete_node(20) # List: 10 -> 15
sll.delete_node(10) # List: 15
sll.delete_node(15) # List: (empty)
sll.get_length() # Length: 0
```

#### Time Complexity for Singly Linked List Operations:

*   **Access/Search**: $O(N)$ - You might have to traverse the entire list.
*   **Insertion**:
    *   At beginning (`prepend`): $O(1)$ - Just update the head pointer.
    *   At end (`append`): $O(N)$ - You need to traverse to the last node.
    *   After a given node (`insert_after`): $O(N)$ to find the node, then $O(1)$ for insertion.
*   **Deletion**:
    *   At beginning: $O(1)$ - Update head.
    *   At end: $O(N)$ - You need to traverse to the second-to-last node to update its `next` pointer.
    *   Of a given node (`delete_node` by value): $O(N)$ to find the node and its predecessor.

---

### 2. Doubly Linked List

A **doubly linked list** is a more sophisticated version where each node contains three fields:
1.  `data`: The actual value stored in the node.
2.  `next`: A pointer or reference to the next node in the sequence.
3.  `prev`: A pointer or reference to the *previous* node in the sequence.

This `prev` pointer allows for traversal in both forward and backward directions, which can be very advantageous for certain operations. The `head` points to the first node, and often a `tail` pointer points to the last node. The `prev` of the head node and the `next` of the tail node are typically `None`.

#### Python Implementation of a Doubly Linked List

First, we need to update our `Node` class to include a `prev` pointer.

```python
class DoublyNode:
    def __init__(self, data):
        self.data = data
        self.next = None  # Reference to the next node
        self.prev = None  # Reference to the previous node
```

Now, let's build the `DoublyLinkedList` class.

```python
class DoublyLinkedList:
    def __init__(self):
        self.head = None
        self.tail = None # Keep track of the tail for O(1) append
        self.length = 0

    def __str__(self):
        """Provides a string representation of the list (forward)."""
        nodes = []
        current = self.head
        while current:
            nodes.append(str(current.data))
            current = current.next
        return " <-> ".join(nodes)

    def append(self, data):
        """Adds a new node with the given data to the end of the list."""
        new_node = DoublyNode(data)
        if not self.head: # List is empty
            self.head = new_node
            self.tail = new_node
        else:
            self.tail.next = new_node # Old tail points to new node
            new_node.prev = self.tail # New node points back to old tail
            self.tail = new_node      # New node becomes the tail
        self.length += 1
        print(f"Appended {data}. List: {self}")

    def prepend(self, data):
        """Adds a new node with the given data to the beginning of the list."""
        new_node = DoublyNode(data)
        if not self.head: # List is empty
            self.head = new_node
            self.tail = new_node
        else:
            new_node.next = self.head # New node points to old head
            self.head.prev = new_node # Old head points back to new node
            self.head = new_node      # New node becomes the head
        self.length += 1
        print(f"Prepended {data}. List: {self}")

    def insert_after(self, prev_data, data):
        """Inserts a new node with 'data' after the node containing 'prev_data'."""
        if not self.head:
            print("List is empty. Cannot insert.")
            return

        current = self.head
        while current and current.data != prev_data:
            current = current.next

        if current is None:
            print(f"Node with data {prev_data} not found.")
            return

        if current == self.tail: # Inserting after the tail
            self.append(data)
            return

        new_node = DoublyNode(data)
        new_node.next = current.next
        new_node.prev = current
        current.next.prev = new_node # The node after 'current' now points back to 'new_node'
        current.next = new_node
        self.length += 1
        print(f"Inserted {data} after {prev_data}. List: {self}")

    def delete_node(self, key):
        """Deletes the first node found with the given 'key' (data)."""
        current = self.head

        while current and current.data != key:
            current = current.next

        if current is None: # Key not found
            print(f"Node with data {key} not found.")
            return

        # Adjust pointers of surrounding nodes
        if current.prev:
            current.prev.next = current.next
        else: # Deleting the head node
            self.head = current.next

        if current.next:
            current.next.prev = current.prev
        else: # Deleting the tail node
            self.tail = current.prev

        # If the list becomes empty after deletion
        if self.head is None:
            self.tail = None

        current = None # Detach the node
        self.length -= 1
        print(f"Deleted {key}. List: {self}")

    def search(self, key):
        """Searches for a node with the given 'key'."""
        current = self.head
        position = 0
        while current:
            if current.data == key:
                print(f"Found {key} at position {position}.")
                return True
            current = current.next
            position += 1
        print(f"{key} not found in the list.")
        return False

    def print_list_forward(self):
        """Prints the list from head to tail."""
        print(f"Forward: {self}")

    def print_list_backward(self):
        """Prints the list from tail to head."""
        nodes = []
        current = self.tail
        while current:
            nodes.append(str(current.data))
            current = current.prev
        print(f"Backward: {' <-> '.join(nodes)}")

    def get_length(self):
        """Returns the current length of the list."""
        print(f"List length: {self.length}")
        return self.length

# --- Doubly Linked List Examples ---
print("\n--- Doubly Linked List Operations ---")
dll = DoublyLinkedList()
dll.append(10) # List: 10
dll.prepend(5) # List: 5 <-> 10
dll.append(20) # List: 5 <-> 10 <-> 20
dll.insert_after(10, 15) # List: 5 <-> 10 <-> 15 <-> 20
dll.print_list_forward() # Forward: 5 <-> 10 <-> 15 <-> 20
dll.print_list_backward() # Backward: 20 <-> 15 <-> 10 <-> 5
dll.delete_node(5) # List: 10 <-> 15 <-> 20 (head deleted)
dll.delete_node(20) # List: 10 <-> 15 (tail deleted)
dll.delete_node(15) # List: 10 (middle deleted)
dll.delete_node(10) # List: (empty)
dll.get_length() # Length: 0
dll.append(1)
dll.append(2)
dll.delete_node(1)
dll.get_length()
```

#### Time Complexity for Doubly Linked List Operations:

*   **Access/Search**: $O(N)$ - Still requires traversal.
*   **Insertion**:
    *   At beginning (`prepend`): $O(1)$ - Update head and its `prev` pointer.
    *   At end (`append`): $O(1)$ - Thanks to the `tail` pointer, we don't need to traverse.
    *   After a given node (`insert_after`): $O(N)$ to find the node, then $O(1)$ for insertion.
*   **Deletion**:
    *   At beginning: $O(1)$ - Update head and the new head's `prev`.
    *   At end: $O(1)$ - Update tail and the new tail's `next`.
    *   Of a given node (`delete_node` by value): $O(N)$ to find the node, then $O(1)$ for deletion because you have direct access to `prev` and `next` nodes.

### Singly vs. Doubly Linked Lists: A Comparison

| Feature | Singly Linked List | Doubly Linked List |
| :------ | :----------------- | :----------------- |
| **Node Structure** | `data`, `next` | `data`, `next`, `prev` |
| **Traversal** | Unidirectional (forward only) | Bidirectional (forward and backward) |
| **Memory Overhead** | Less (one pointer per node) | More (two pointers per node) |
| **Insertion at End** | $O(N)$ (requires traversal) | $O(1)$ (with `tail` pointer) |
| **Deletion of a Node** | $O(N)$ (requires finding predecessor) | $O(N)$ to find node, then $O(1)$ to delete (direct access to `prev` and `next`) |
| **Complexity** | Simpler to implement | More complex to implement (more pointers to manage) |

### When to Use Linked Lists?

Linked lists, despite their $O(N)$ access time, are incredibly useful in specific scenarios:

*   **Dynamic Data**: When the number of elements is not known beforehand or changes frequently (e.g., managing a playlist, browser history).
*   **Frequent Insertions/Deletions**: If you're constantly adding or removing elements from the middle of a collection, linked lists can outperform arrays (which might require shifting many elements).
*   **Implementing Other Data Structures**: They are fundamental to building stacks, queues, hash maps (for collision resolution), and even more complex structures like graphs.
*   **Memory Management**: In low-level programming, linked lists can be used to manage free memory blocks.

### Conclusion

Linked lists are a powerful and flexible data structure that every developer should master. They offer distinct advantages over arrays, particularly in scenarios involving dynamic data and frequent modifications. While singly linked lists are simpler, doubly linked lists provide greater flexibility for traversal and deletion at the cost of slightly more memory and implementation complexity.

Understanding their underlying mechanics, the trade-offs involved, and their appropriate use cases will significantly enhance your problem-solving toolkit. Keep practicing, and don't hesitate to experiment with these concepts!