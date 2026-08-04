# What is React?

## Definition

React is a **JavaScript library for building user interfaces (UI)**. Instead of manually changing HTML whenever data changes, React lets you describe **what the UI should look like for a given state**, and it updates the page automatically.

Think of React as a tool that keeps your UI and your data in sync.

---

## Why was React created?

Imagine you're building a shopping cart using plain JavaScript.

When a user adds an item, you need to:

* Create new HTML elements
* Update the cart count
* Update the total price
* Enable/disable buttons
* Remove elements if items are deleted

As the application grows, managing all these manual DOM updates becomes difficult and error-prone.

React solves this by letting you focus on **updating data**, while React figures out **how the UI should change**.

---

## React's Core Idea

In React, the UI is a **function of your data (state).**

Instead of saying:

> "Change this button text."

You say:

> "If the user is logged in, show Logout. Otherwise, show Login."

Example:

```jsx
function App() {
  const loggedIn = true;

  return (
    <button>
      {loggedIn ? "Logout" : "Login"}
    </button>
  );
}
```

When `loggedIn` changes, React automatically updates the button.

---

## Component-Based Architecture

React applications are built using **components**.

A component is a reusable piece of UI that has its own logic and appearance.

Instead of creating one huge HTML page, you split it into smaller parts.

Example:

```
App
├── Navbar
├── Sidebar
├── ProductList
│   ├── ProductCard
│   ├── ProductCard
│   └── ProductCard
└── Footer
```

Each component can be developed, tested, and reused independently.

Example:

```jsx
function Button() {
  return <button>Click Me</button>;
}
```

Now you can use it multiple times:

```jsx
<Button />
<Button />
<Button />
```

---

## Declarative Programming

React uses a **declarative** approach.

Instead of writing steps to update the UI, you describe what the UI should look like.

Imperative (Plain JavaScript):

```javascript
const button = document.getElementById("btn");
button.innerText = "Loading...";
button.disabled = true;
```

Declarative (React):

```jsx
<button disabled={loading}>
  {loading ? "Loading..." : "Submit"}
</button>
```

You change the `loading` variable, and React updates the UI.

---

## JSX

React uses **JSX**, which looks like HTML but is actually JavaScript syntax.

Example:

```jsx
const element = <h1>Hello React!</h1>;
```

JSX makes UI code easier to read because the structure and logic are written together.

You can also use JavaScript inside JSX.

```jsx
const name = "John";

return <h1>Hello {name}</h1>;
```

---

## State

State is data that **can change over time**.

Examples:

* Counter value
* Form input
* Logged-in user
* Theme (Dark/Light)
* Shopping cart

Example:

```jsx
const [count, setCount] = useState(0);
```

When state changes, React re-renders the component to reflect the new data.

---

## Props

Props are values passed from one component to another.

Parent:

```jsx
<UserCard name="Alice" />
```

Child:

```jsx
function UserCard(props) {
  return <h2>{props.name}</h2>;
}
```

Props are **read-only**. A child component should not modify them.

---

## Virtual DOM

The browser has a **DOM (Document Object Model)** that represents the webpage.

Updating the DOM directly can become expensive if done frequently.

React uses a **Virtual DOM**, which is a lightweight JavaScript representation of the real DOM.

When state changes:

1. React creates a new Virtual DOM.
2. Compares it with the previous one.
3. Finds only what changed.
4. Updates only those parts in the real DOM.

This process is called **Reconciliation**.

Example:

Before:

```html
<h1>Count: 0</h1>
```

After state changes:

```html
<h1>Count: 1</h1>
```

React updates only the text instead of rebuilding the whole page.

---

## One-Way Data Flow

Data in React flows **from parent to child**.

```
App
 ↓
Navbar
 ↓
Profile
 ↓
Avatar
```

A child receives data through props.

If a child wants to change parent data, it calls a function provided by the parent.

This makes applications easier to understand and debug.

---

## React Doesn't Replace JavaScript

React is **not** a programming language.

It is a JavaScript library.

You still use:

* Variables
* Arrays
* Objects
* Functions
* Loops
* Conditions
* Async/Await

React simply gives you a better way to build interfaces.

---

## Simple Example

```jsx
function App() {
  const [count, setCount] = useState(0);

  return (
    <>
      <h1>{count}</h1>

      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </>
  );
}
```

What happens:

* Initial count is `0`
* User clicks the button
* `setCount()` updates the state
* React re-renders the component
* UI now displays `1`

Notice that you never manually update the `<h1>`. React does it for you.

---

# Why Developers Like React

* Reusable components
* Easier to maintain large applications
* Automatic UI updates
* Predictable data flow
* Huge ecosystem and community
* Works well for Single Page Applications (SPAs)

---

# Common Interview Questions

### Is React a framework?

No. React is a **JavaScript library** focused on building user interfaces. Frameworks like Next.js are built on top of React and provide additional features such as routing, server-side rendering, and API handling.

---

### Why is React called a library?

Because it solves one main problem—building UIs. It doesn't force a complete project structure or include everything needed for an application. You choose additional libraries (or a framework like Next.js) for routing, state management, and other features.

---

### What is React mainly responsible for?

React handles:

* Rendering UI
* Updating UI when data changes
* Managing component state
* Efficient DOM updates

---

### Does React use HTML?

No. React uses **JSX**, which looks like HTML but is JavaScript syntax that gets converted into JavaScript function calls during the build process.

---

### Does React manipulate the DOM directly?

Not usually. React updates its Virtual DOM first, compares it with the previous version, and then applies the minimum required changes to the real DOM.

---

# Key Takeaways

* React is a JavaScript library for building user interfaces.
* It uses reusable **components** to organize applications.
* It follows a **declarative** approach: describe the UI instead of manually updating it.
* **State** holds changing data, while **props** pass data between components.
* The **Virtual DOM** helps React update the real DOM efficiently.
* React uses **one-way data flow**, making applications more predictable.
* React focuses on the UI layer; tools like **Next.js** build on top of React to provide a complete application framework.
