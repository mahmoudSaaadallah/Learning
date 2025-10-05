### What are Union Types?

At its core, a **Union Type** allows a variable, parameter, or return value to be one of several specified types. You declare a union type using the vertical bar (`|`) between the types. It's like saying, "this can be *either* type A *or* type B *or* type C."

This is distinct from `any` (which opts out of type checking) or `unknown` (which requires explicit type narrowing before use). Union types provide a precise way to express that a value can legitimately belong to a finite set of types, and TypeScript will enforce that.

### Syntax

The syntax is straightforward:

```typescript
type MyFlexibleType = TypeA | TypeB | TypeC;

let value: string | number;
value = "hello"; // Valid
value = 123;     // Valid
// value = true; // Error: Type 'boolean' is not assignable to type 'string | number'.

function processInput(input: string | number | boolean): void {
  // ...
}
```

### Why Use Union Types? (Benefits)

1.  **Flexibility with Type Safety**: You get the flexibility of accepting multiple types without sacrificing the benefits of static type checking. The compiler ensures that only the specified types are used.
2.  **Improved Readability**: The type signature clearly communicates the expected range of values.
3.  **Enhanced IDE Support**: Autocompletion and refactoring tools become more intelligent, understanding the potential types and their respective properties.
4.  **Type Narrowing**: This is where union types truly shine. TypeScript's control flow analysis can "narrow" a union type to a more specific type within a conditional block, allowing you to safely access type-specific properties and methods.

### Type Narrowing with Union Types

When you have a union type, you can only access properties or methods that are common to *all* types in the union. To access type-specific members, you need to narrow the type. TypeScript provides several mechanisms for this:

#### 1. `typeof` Type Guards (for Primitives)

This is ideal for distinguishing between primitive types like `string`, `number`, `boolean`, `symbol`, `bigint`, `undefined`, and `object`.

```typescript
function printId(id: string | number): void {
  if (typeof id === "string") {
    // 'id' is now narrowed to 'string'
    console.log(id.toUpperCase());
  } else {
    // 'id' is now narrowed to 'number'
    console.log(id.toFixed(2));
  }
}

printId("abc-123"); // ABC-123
printId(456.789);   // 456.79
```

#### 2. `instanceof` Type Guards (for Classes/Objects)

Useful for distinguishing between different class instances.

```typescript
class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    // 'animal' is narrowed to 'Dog'
    animal.bark();
  } else {
    // 'animal' is narrowed to 'Cat'
    animal.meow();
  }
}

makeSound(new Dog()); // Woof!
makeSound(new Cat()); // Meow!
```

#### 3. Property Presence Checks (`in` Operator)

When dealing with object types that might have different sets of properties, the `in` operator is very effective.

```typescript
interface Car {
  drive(): void;
  brand: string;
}

interface Boat {
  sail(): void;
  length: number;
}

function operateVehicle(vehicle: Car | Boat): void {
  if ("drive" in vehicle) {
    // 'vehicle' is narrowed to 'Car'
    vehicle.drive();
    console.log(`Driving a ${vehicle.brand} car.`);
  } else {
    // 'vehicle' is narrowed to 'Boat'
    vehicle.sail();
    console.log(`Sailing a ${vehicle.length}m boat.`);
  }
}

const myCar: Car = { drive: () => console.log("Vroom"), brand: "Tesla" };
const myBoat: Boat = { sail: () => console.log("Whoosh"), length: 10 };

operateVehicle(myCar);  // Vroom, Driving a Tesla car.
operateVehicle(myBoat); // Whoosh, Sailing a 10m boat.
```

#### 4. Discriminated Unions (The Most Powerful for Complex Objects)

This is a highly effective pattern for working with unions of object types that share a common, literal property (the "discriminant"). This property's value then determines the specific type within the union.

```typescript
interface SuccessResult {
  status: "success"; // Discriminant property
  data: any;
}

interface ErrorResult {
  status: "error"; // Discriminant property
  message: string;
  errorCode: number;
}

type APIResult = SuccessResult | ErrorResult;

function handleResult(result: APIResult): void {
  if (result.status === "success") {
    // 'result' is narrowed to 'SuccessResult'
    console.log("Data received:", result.data);
  } else {
    // 'result' is narrowed to 'ErrorResult'
    console.log(`Error ${result.errorCode}: ${result.message}`);
  }
}

const success: APIResult = { status: "success", data: { user: "Alice" } };
const error: APIResult = { status: "error", message: "Not Found", errorCode: 404 };

handleResult(success); // Data received: { user: 'Alice' }
handleResult(error);   // Error 404: Not Found
```

Discriminated unions are particularly useful in state management (e.g., Redux actions, React component states) where an object's `type` property often serves as the discriminant.

### Common Use Cases

*   **Function Parameters**: A function that can accept different types of input (e.g., `function log(message: string | number) { ... }`).
*   **Return Types**: A function that might return different types based on its execution path (e.g., `function fetchData(): User | Error { ... }`).
*   **State Management**: Representing different states of an entity (e.g., `type LoadingState = { status: "loading" } | { status: "success", data: any } | { status: "error", error: string };`).
*   **API Responses**: Handling varied response structures from an API.

### Conclusion

Union Types are an indispensable feature in TypeScript for senior developers. They allow you to write more robust, flexible, and maintainable code by precisely defining the possible types a value can take, while still leveraging TypeScript's powerful type-checking and narrowing capabilities. Mastering them, especially in conjunction with type guards and discriminated unions, is key to writing high-quality, type-safe TypeScript applications.