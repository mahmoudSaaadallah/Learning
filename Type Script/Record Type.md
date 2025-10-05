The Record generic type is useful when creating an object, but we don't know the exact properties that we will use in this object, or the exact values for those properties.

### What is the `Record` Type?

The `Record<Keys, Type>` utility type is a generic type that constructs an object type whose property keys are `Keys` and whose property values are `Type`.

Essentially, it's a shorthand for defining an object where:
1.  All property **keys** must conform to a specific type (e.g., `string`, `number`, `symbol`, or a union of literal strings/numbers).
2.  All property **values** must conform to another specific type.

It's particularly useful for creating "dictionary" or "map-like" object types where you know the structure of the keys and the values, but not necessarily the exact names of all keys upfront.

### Syntax

The syntax is `Record<Keys, Type>`:

*   `Keys`: A union of `string | number | symbol` or a literal type (e.g., `'id' | 'name' | 'email'`). This defines the type of the property names (keys).
*   `Type`: Any TypeScript type. This defines the type of the property values.

### How it Works (Under the Hood)

`Record` is implemented using mapped types. Conceptually, it looks something like this:

```typescript
type Record<K extends keyof any, T> = {
  [P in K]: T;
};
```
This means for every `P` (property) in `K` (Keys), that property will have the type `T`.

```TypeScript
let data:Record<string, string | number>;
data = {
	name:"mahmoud", 
	age: 25
}
// or 
date = {
	email:"mahmoud.saadallah73@gmail.com"
}
// or
data = {
	'val':12.3
}
```

- In this example we specify the type of the data variable to be object of Records which each record is key-value pair. The key must be string and the value could be either string or number.
- we have to know that when using `name:"Mahmod"` like this this mean the key is type of string as the JavaScript consider the key as it's surrounded by double quotes. `"name":"Mahmoud"`.

### Why Use `Record`? (Benefits and Use Cases)

1.  **Type-Safe Dictionaries/Maps**: This is the primary use case. When you need an object where you know the type of the keys and the type of the values, but the specific keys might be dynamic or too numerous to list explicitly.

    ```typescript
    // Example: A dictionary mapping user IDs (string) to User objects
    interface User {
      id: string;
      name: string;
      email: string;
    }

    type UserDictionary = Record<string, User>;

    const users: UserDictionary = {
      "user-123": { id: "user-123", name: "Alice", email: "alice@example.com" },
      "user-456": { id: "user-456", name: "Bob", email: "bob@example.com" },
    };

    // users["user-789"] = { id: "user-789", name: "Charlie", email: "charlie@example.com" }; // Valid
    // users["user-abc"] = { id: "user-abc", name: "David", age: 30 }; // Error: 'age' does not exist in type 'User'
    ```

2.  **Enforcing Uniform Value Types**: When you have a fixed set of keys, but you want to ensure all their values are of a particular type.

```typescript
    // Example: Configuration settings where all values are strings
    type AppConfigKeys = 'theme' | 'language' | 'apiUrl';
    type AppConfig = Record<AppConfigKeys, string>;

    const config: AppConfig = {
      theme: 'dark',
      language: 'en-US',
      apiUrl: 'https://api.example.com',
      // version: '1.0.0' // Error: Object literal may only specify known properties, and 'version' does not exist in type 'AppConfig'.
    };

    // config.theme = 123; // Error: Type 'number' is not assignable to type 'string'.
    ```

3.  **Creating Lookup Tables**: Similar to dictionaries, but often for more static data.

    ```typescript
    enum Status {
      Active = 'ACTIVE',
      Inactive = 'INACTIVE',
      Pending = 'PENDING',
    }

    type StatusDisplay = Record<Status, string>;

    const statusLabels: StatusDisplay = {
      [Status.Active]: 'Currently Active',
      [Status.Inactive]: 'Not Active',
      [Status.Pending]: 'Awaiting Approval',
    };

    console.log(statusLabels[Status.Active]); // "Currently Active"
    ```
    Notice how `Record<Status, string>` ensures that *all* enum members are present as keys and their values are strings. If you miss one, TypeScript will complain.

4.  **Partial/Optional Records**: You can combine `Record` with other utility types like `Partial` or `Readonly`.

    ```typescript
    type OptionalUserConfig = Partial<Record<string, string>>;

    const userSettings: OptionalUserConfig = {
      'notifications': 'enabled',
      'darkMode': 'true',
    };

    // userSettings.invalidKey = 123; // Error: Type 'number' is not assignable to type 'string'.
    ```

### `Record` vs. Other Object Type Definitions

*   **`interface` or `type` alias for explicit objects**:
    ```typescript
    interface UserMap {
      [key: string]: User; // Index signature
    }
    type UserMapAlias = {
      [key: string]: User; // Index signature
    }
    ```
    `Record<string, User>` is essentially equivalent to an object type with a string index signature (`{ [key: string]: User; }`). The main advantage of `Record` is its conciseness and expressiveness, especially when the key type is a union of literals or an enum. It's often preferred for its functional style.

*   **`Map<K, V>`**:
    *   `Map` is a **runtime data structure** in JavaScript. It's an actual object instance that you create (`new Map()`).
    *   `Record` is a **compile-time type definition**. It describes the shape of a plain JavaScript object (`{}`).
    *   Choose `Map` when you need the specific methods and behavior of a `Map` (e.g., `set()`, `get()`, `has()`, `size`, iteration order guarantees, keys can be any value, not just strings/symbols/numbers).
    *   Choose `Record` when you're defining the type of a plain JavaScript object that acts like a dictionary.

    ```typescript
    // Record (compile-time type for plain object)
    type UserRecord = Record<string, User>;
    const userObj: UserRecord = { /* ... */ };

    // Map (runtime data structure)
    const userMap: Map<string, User> = new Map();
    userMap.set("user-123", { id: "user-123", name: "Alice", email: "alice@example.com" });
    ```

### Practical Considerations

*   **Key Types**: The `Keys` argument for `Record` must be assignable to `string | number | symbol`. This means you can't use arbitrary objects as keys, which is consistent with plain JavaScript object keys.
*   **Readability**: For simple, fixed-key objects, a direct interface or type alias might be more readable. `Record` shines when the key set is dynamic, derived from an enum, or a union of many literal strings.
*   **Immutability**: Remember that `Record` defines the *shape* of an object. The object itself is mutable by default. If you need immutability, combine it with `Readonly<T>`:
    ```typescript
    type ReadonlyUserDictionary = Readonly<Record<string, User>>;
    const immutableUsers: ReadonlyUserDictionary = { /* ... */ };
    // immutableUsers["user-123"] = { /* ... */ }; // Error: Cannot assign to 'user-123' because it is a read-only property.
    ```

### Conclusion

The `Record` utility type is a powerful tool in a senior TypeScript developer's arsenal. It provides a clean, type-safe, and expressive way to define object types that function as dictionaries or maps, where the keys and values adhere to specific types. Understanding when to reach for `Record` versus a traditional interface or a `Map` instance is crucial for writing idiomatic, maintainable, and robust TypeScript applications.