# Week 6: Introduction to React – Building Interactive UIs with Vite

## Introduction

We’re going to learn about **React**, a powerful tool for building interactive web pages. We’ll set up a React app using **Vite** (the same tool from Week 5) and create a simple interactive form. This guide is written for someone new to coding (like a 10-year-old), so we’ll explain everything step by step. You’ll use the Windows Command Prompt, just like in previous weeks, to set up and run your project. By the end, you’ll have a web page where you can type in your name and age, and it will instantly display a personalized message (including whether you can vote and a pretend train fare). Let’s get started!

### What You’ll Learn

* **React**: A JavaScript library for building user interfaces (UIs). Think of it like Lego blocks for web pages – you build big pages out of small pieces.
* **Components**: The reusable building blocks in React. Each component is like a mini-webpage (a button, a form, a header) that you can mix and match.
* **JSX**: A special syntax that lets us write HTML inside JavaScript. It’s what React uses to describe UIs, and Vite helps translate it into normal JavaScript that browsers understand.
* **Props**: Short for “properties,” these are pieces of data you can pass into components. It’s like giving a component a suitcase of information to use (for example, a name or a color).
* **State (useState)**: A way for components to remember things. State is like a component’s own memory – when it changes, the UI updates automatically to reflect the new data.
* **Events and onChange**: How we handle user inputs (like typing in a text box) in React. We’ll use an event handler to update state whenever you type, creating an interactive experience.

### What You’ll Do

1. **Exercise A**: Set up Node.js and Vite, create a new React project, and run a “Hello World” React app. Then change it to say **“Hello React World with Vite!”** in your browser.
2. **Exercise B**: Create a new React component called `Greeting.jsx`. Learn how to pass **props** (like name, age, and color) to this component, and use **destructuring** to access them easily. We’ll also use **inline CSS styles** via props to make our greeting colorful.
3. **Exercise C**: Add interactivity with **state** and **events**. Use React’s `useState` hook to store a name and age. Build a simple form (text input for name and number input for age) with `onChange` events so that as you type, the page updates. We’ll show a personalized greeting, a voting eligibility message (using a **ternary operator** for conditional rendering), and a calculated train fare, all updating instantly as you input values.
4. **Save Everything**: Keep your work organized in a folder and (optionally) save it to GitHub so you don’t lose it.

### Prerequisites

* You’ve done Weeks 1–5, so you have some experience with Node.js and have Node.js installed. (Check by typing `node --version` in Command Prompt.)
* You know how to open the Command Prompt and run basic commands (`cd`, `mkdir`, etc. from earlier weeks).
* You understand basic JavaScript and HTML from previous exercises (though we’ll explain new concepts as we go).
* Ideally, you completed Week 5, so you have Vite installed or at least know how to use it (we’ll cover setup again briefly). If Node.js isn’t installed, download it from [nodejs.org](https://nodejs.org) and install it before starting.

## Step 1: Understanding the Basics

Before we start coding, let’s learn what React is and how it differs from the plain HTML/JS websites you’ve built before. This is like learning the rules of a new game before playing.

### What is React?

**React** is a popular JavaScript library (created by developers at Facebook) for building **user interfaces**. In simpler terms, React helps you make websites that **change dynamically** as users interact with them, without having to reload the page each time.

Think of building a website with React like playing with Lego bricks:

* In normal HTML/JavaScript, if you want to update part of a page (say, to show new information after a user clicks a button), you have to manually change the HTML using JavaScript (for example, by `document.getElementById(...).innerText = "new text"`). This is like having to redraw or adjust parts of a picture by hand whenever something changes.
* In React, you describe what the UI should look like **given a certain state of data**, and React will handle updating the browser for you when that data changes. React keeps a sort of *blueprint* of the DOM (the page structure) in memory (often called the **virtual DOM**). When something changes, React compares the new blueprint to the old one and updates only the parts that changed. This makes it efficient and easier to manage. It’s like telling a helper, “Whenever my data changes, make the necessary small changes to the display,” instead of repainting the whole picture yourself.

In summary, **React = JavaScript + a smart updating system**. You write components (in JSX, explained below) and define how they should look based on some data, and React takes care of inserting, updating, or removing the right elements on the actual webpage.

### How is React Different from Regular HTML/JS?

With regular HTML and JavaScript, your web pages are mostly static unless you use JavaScript to change them. If you have multiple parts of the page that need to change, you end up writing a lot of code to query the DOM and update each element. For example, imagine a simple page that greets a user by name: without React, you might have to grab a `<span>` and set its text content every time the user types their name in an input box.

**React simplifies this by using a declarative approach**:

* *Declarative* means you just declare *what* you want the UI to look like for each state of your data. For instance, “If the user’s name is X, I want the page to show ‘Hello X’.” You don’t explicitly write the steps to update the page every time—React figures that out.
* In contrast, the traditional *imperative* approach (regular JS) means writing out the exact steps: “When the input changes, get the `<span>` and set its text to ...”.

Also, in React you break your UI into **components** (next section), which makes it easier to reuse parts of the page and reason about each part in isolation. In regular HTML/JS, you might copy-paste code or juggle lots of global variables; in React, each component can manage its own state and logic, which is like each part of your interface having its own little brain.

Lastly, React uses **JSX** and needs a build tool (like Vite) to convert that JSX into normal JavaScript and HTML for the browser. This sounds complex, but tools like Vite make it seamless. You’ll write code that looks like HTML inside your JavaScript, and Vite will magically turn it into code that can run in any browser.

### Components

A **component** in React is like a reusable piece of the user interface. It could be as small as a button or as big as an entire form. You can think of components as custom HTML elements that you define. For example, you might create a `<Greeting>` component that always displays “Hello, [Name]!” given a name.

Components are usually written as JavaScript functions (or classes, but we will use functions for simplicity). A component function returns JSX (the UI structure for that component). For instance, a very simple component might look like this (don’t worry about the exact syntax yet):

```jsx
function Hello() {
  return <h1>Hello there!</h1>;
}
```

This defines a component `Hello` that, when used, renders an `<h1>` heading. To use this component in a React app, you could include `<Hello />` in another component’s JSX, and React will display “Hello there!” at that spot.

Components let us break a complex UI into smaller, manageable pieces. Each component can also **receive data** and display different things based on that data – that’s where **props** come in.

### Props (Properties)

**Props** are the way we pass data into components. Think of props as function arguments, or like customizing a toy by giving it different accessories. For example, if you have a `Greeting` component, you might want it to greet different people by name. You could design `Greeting` to accept a prop called `name`. Then you can use the component like `<Greeting name="Alice" />` or `<Greeting name="Bob" />`, and it would display “Hello, Alice!” or “Hello, Bob!” respectively.

In code, inside the `Greeting` component function, you would receive `props` (often by destructuring, which we’ll show later) and use `props.name` to get the name passed in.

Props are **read-only** – they are passed *into* a component and the component should not modify them (just like how a function shouldn’t change the arguments you pass to it). If a component needs to remember or change data, that’s when we use **state**.

### State and useState

**State** is a built-in concept in React that allows components to create and manage their own data that can change over time. Unlike props, which come from outside, **state is internal** to the component. When state changes, React will re-run the component function and update the UI to reflect those changes. This is one of React’s superpowers: you change the data, React updates the display.

For example, think of a counter component. It might have a state variable `count`. When you click a “+” button, the component updates `count`, and React will re-render the component to show the new count.

React provides a function called `useState` (we call it a *hook*) to let components use state. We’ll use `useState` in our code soon. It gives us two things: the current value of the state, and a function to change it. For instance:

```jsx
const [count, setCount] = useState(0);
```

This line means: “Let there be a state variable `count`, starting at `0`. And provide a function `setCount` to update `count`.” If we call `setCount(5)`, React will set `count` to 5 **and** automatically re-render the component to reflect this new value in the UI.

**Key idea:** State changes trigger re-renders. That means if we connect state to what’s displayed in the JSX, updating the state will automatically update what you see on the page. We don’t have to manually manipulate the DOM – React does it for us.

### JSX

**JSX** stands for JavaScript XML. It’s the syntax React uses to describe the UI. JSX lets you write code that looks like HTML inside your JavaScript. For example:

```jsx
return <h1>Hello React!</h1>;
```

This looks like HTML, but under the hood it’s JavaScript. The browser doesn’t understand JSX directly; that’s why we use a bundler like Vite. **Vite will transform JSX into regular JavaScript** that creates the HTML elements. The `.jsx` file extension is used for files that contain JSX so that the build tool knows to process them appropriately.

JSX makes it easy to visualize the structure of your UI in code, and you can embed dynamic values with curly braces `{ }`. For instance, if you have a variable `name` set to `"Ada"`, you can write:

```jsx
<h2>Hello {name}!</h2>
```

If `name` is `"Ada"`, this will render as `<h2>Hello Ada!</h2>` in the HTML. The `{ }` lets you drop JavaScript values or expressions into your JSX.

We’ll be writing JSX in our React components. Don’t worry if it feels a bit strange at first (mixing HTML-looking syntax in JS); you’ll get used to it with practice. Just remember that under the hood, it’s all JavaScript – JSX is converted to `React.createElement` calls which create the actual DOM elements.

### Putting It Together

* **React** helps us manage complex UIs by breaking them into components and updating them efficiently.
* We use **components** (with JSX) to declare what our UI should look like.
* We pass data with **props** into components, and components manage changing data with **state**.
* React + Vite will handle converting our JSX and making everything work in the browser with live reloading as we edit (thanks to Vite’s development server).

Now that we have a basic idea of the concepts, let’s get our hands dirty by setting up the project.

## Step 2: Set Up Your Project

Let’s create a new folder for this week’s project and set up a React app using Vite. This is like preparing our workspace and tools before building something.

1. **Open Command Prompt**:

   * Press `Win + R`, type `cmd`, and press Enter. This opens the black Command Prompt window.
   * First, check that Node.js and npm are installed (they should be, if you did previous weeks):

     ```bash
     node --version
     npm --version
     ```

     * You should see version numbers (e.g., `v18.x.x` for Node, and a similar version for npm). If you get an error or no version, install Node.js from the official website [nodejs.org](https://nodejs.org) and then reopen the Command Prompt.

2. **Create a Project Folder**:

   * Navigate to your Desktop (or wherever you keep your WAD projects):

     ```bash
     cd %userprofile%\Desktop
     ```

     * This changes directory to your desktop. It’s like moving to the folder where we want to create our project.
   * Make a new folder for Week 6 and enter it:

     ```bash
     mkdir WAD-week6
     cd WAD-week6
     ```

     * `mkdir` creates a folder (named `WAD-week6` here), and `cd` moves you inside that folder, like opening a new workspace.

3. **Initialize a Vite + React Project**:

   * We’ll use Vite’s project generator to create a React template. In the `WAD-week6` folder, run:

     ```bash
     npm create vite@latest . -- --template react
     ```

     * **Breakdown**: This command runs the Vite project creation script.

       * `vite@latest` means we’re using the latest version of Vite’s initializer.
       * `.` (a single dot) tells it to set up the project in the current directory (WAD-week6) instead of making a new subfolder. It will use the current folder name as the project name.
       * `--template react` tells Vite to use the **React template** (since Vite can scaffold for other frameworks too).
     * After running this, Vite will generate a bunch of files and folders for a React app. You might see prompts or progress messages. If prompted to select a framework or variant and you *didn’t* use the exact command above, choose **React** and **JavaScript** (not TypeScript) for this tutorial.
     * When it’s done, run:

       ```bash
       dir
       ```

       You should see files like `package.json`, `index.html`, and folders like `src`. Vite has created these for you:

       * **`package.json`** – contains project info and dependencies (you’ll see React and ReactDOM listed here).
       * **`index.html`** – the main HTML file that includes your React app.
       * **`src` folder** – contains the source code (you’ll find `main.jsx` and `App.jsx` already made).
       * **`node_modules`** – (this might not be there yet; we’ll create it in the next step by installing dependencies.)

4. **Install Dependencies**:

   * If the previous step didn’t already install the project’s dependencies (it usually doesn’t automatically), you need to install them. In Command Prompt, run:

     ```bash
     npm install
     ```

     * This tells npm to install the packages listed in `package.json` (React, ReactDOM, etc.) into a new `node_modules` folder. It may take a few moments and print a lot of lines.
     * When done, run `dir` again and you should now see a `node_modules` folder. This folder contains all the code libraries your app needs (React, Vite, etc.). It’s like the pantry with all the ingredients for your project.

5. **Open the Project in Code (Optional)**:

   * You can use Notepad via the command line to edit files (we will do that for specific files soon). If you have a code editor like VS Code installed, you could also open the whole `WAD-week6` folder there for a nicer experience. But we’ll assume you stick to Notepad for now as in previous weeks.

Now our project is set up with React and Vite! Next, let’s run the development server and see what the default React app looks like, then customize it.

## Step 3: Exercise A – Hello React World

Our first coding exercise is to get the default React app running and then make it display a custom message. This will verify that our setup works and introduce us to editing React components.

### A.1: Run the Default React App

1. **Start the Vite Development Server**:

   * In the Command Prompt (still in the `WAD-week6` directory), run:

     ```bash
     npx vite dev
     ```

     * This starts Vite’s dev server on a local address (usually `http://localhost:5173`). You should see output like:

       ```
       VITE vX.X.X  ready in X ms
       ➜  Local:   http://localhost:5173/
       ```
     * *Note:* We use `npx vite dev` here just like in Week 5. This runs Vite from our project’s dependencies. (Alternatively, we could run `npm run dev` which does the same thing because Vite set up a script for us, but we’ll stick to `npx` for consistency.)

2. **View in the Browser**:

   * Open your web browser (Chrome, etc.) and go to **[http://localhost:5173](http://localhost:5173)**. You should see a page that says **“Vite + React”** with a spinning React logo and a counter button that increments when clicked. This is the default template that Vite created for us.
   * Congratulations, you have a React app running! The page you see is being served by Vite and is using React under the hood.

3. **Hot Module Reloading (HMR) Test**:

   * Vite has **Hot Module Reloading**, which means as soon as you save changes to your code, the browser updates automatically without a full reload.
   * Let’s do a quick test: The page likely says “Edit `src/App.jsx` and save to test HMR” at the bottom. We will do just that in the next section by modifying the App component. Keep your browser open at `localhost:5173` and side-by-side with the editor if possible, so you can see changes live.

### A.2: Update the Welcome Message to "Hello React World with Vite!"

The default page isn’t very personal. Let’s change the heading to say “Hello React World with Vite!” as a simple first edit.

1. **Open `src/App.jsx` in Notepad**:

   * Go back to the Command Prompt (if you stopped Vite, you can press `Ctrl+C` to stop it; but you can actually leave it running and open a new Command Prompt window for editing, since Vite will detect changes as you save).
   * Open the file with Notepad:

     ```bash
     notepad src\App.jsx
     ```
   * This will open the `App.jsx` file in Notepad. This file defines the main App component of our React app.

2. **Understand `App.jsx` (briefly)**:

   * You’ll see something like:

     ```jsx
     import { useState } from 'react'
     import reactLogo from './assets/react.svg'
     import viteLogo from '/vite.svg'
     import './App.css'

     function App() {
       const [count, setCount] = useState(0)

       return (
         <div className="App">
           ... a bunch of HTML-like JSX ...
         </div>
       )
     }

     export default App
     ```

     This is the default content Vite gave us. There’s a lot here – imports of logos, some CSS, a `useState` for a counter, etc. We don’t need all of it for our purposes.

3. **Simplify and Change the JSX**:

   * We will edit this file to display a simple greeting. Replace the contents of the return statement with a single heading. For example, change the `return (...)` part to:

     ```jsx
       return (
         <h1>Hello React World with Vite!</h1>
       );
     ```

     and remove any other JSX inside the return. Also, since we won’t need the counter or logos, you can remove or ignore these lines:

     * Delete the lines that import `reactLogo` and `viteLogo`, and the line `import './App.css'` (to simplify things).
     * Delete the line with `const [count, setCount] = useState(0)` (we will use `useState` later, but not for this counter).
     * Ensure your App function now simply returns the `<h1>...</h1>` element.

   * Your edited `App.jsx` might now look like:

     ```jsx
     import React from 'react';

     function App() {
       return (
         <h1>Hello React World with Vite!</h1>
       );
     }

     export default App;
     ```

   * Save the file (File > Save in Notepad), and close Notepad.

4. **See the Changes**:

   * If your Vite dev server is still running, you should see the browser auto-update. The page at `localhost:5173` will now just show a big **“Hello React World with Vite!”** message on a white background.
   * If you stopped the dev server earlier, restart it with `npx vite dev` and refresh the browser. You should see the updated message.
   * Congratulations, you’ve edited your first React component! 🎉

5. **Why This Matters**:

   * This simple change shows the power of Vite’s Hot Module Reloading (HMR) — as soon as you saved `App.jsx`, the browser updated the content to reflect your changes (no manual refresh needed).
   * You also got a taste of JSX by writing an `<h1>` inside a React component. This mix of HTML and JS is what JSX is all about.
   * From now on, we will build on this App component and add more features. But always remember: if something isn’t showing up or you see an error, check the Command Prompt where Vite is running and the browser console (press F12 in the browser) for error messages. React will often give helpful warnings there.

Now that our app greets the whole world, let’s make it more personal by creating a new component for personalized greetings.

## Step 4: Exercise B – Creating a Component and Using Props

In this exercise, we will create a new component called `Greeting` that can greet a person by name (and even mention their age). We’ll learn how to pass props to the component and how to use **destructuring** to easily use those props inside the component. We’ll also demonstrate inline styling via props (for example, to color the text).

### B.1: Create `Greeting.jsx` Component

1. **Create a new file for the component**:

   * In the Command Prompt, create a new file in the `src` folder for our component:

     ```bash
     notepad src\Greeting.jsx
     ```
   * This will open a blank Notepad window for a new file `Greeting.jsx`.

2. **Write the Greeting component code**:

   * In `Greeting.jsx`, type the following and then we’ll explain it:

     ```jsx
     import React from 'react';

     function Greeting({ name, age, color }) {
       return (
         <div>
           <h2 style={{ color: color }}>Hello {name}!</h2>
           <p>You are {age} years old.</p>
         </div>
       );
     }

     export default Greeting;
     ```
   * Save the file (File > Save) and leave it open or close Notepad as you prefer.

3. **Understanding the code**:

   * `import React from 'react';`: This imports React. (With modern React this import isn’t strictly required for JSX to work, but it doesn’t hurt and can clarify where JSX comes from. We include it here for clarity.)
   * `function Greeting({ name, age, color }) { ... }`: Here we define a new React component named `Greeting`. It’s a function that takes one argument – an object of props. We use **destructuring** in the function parameter to directly pull out `name`, `age`, and `color` from the props object. This is equivalent to writing:

     ```jsx
     function Greeting(props) {
       const name = props.name;
       const age = props.age;
       const color = props.color;
       ...
     }
     ```

     but it’s shorter and clearer. We now have variables `name`, `age`, and `color` to use inside our component.
   * `return ( <div> ... </div> );`: The component returns a JSX structure. In this case, we wrap everything in a `<div>` (a container) because JSX requires a single root element to return. Inside, we have:

     * `<h2 style={{ color: color }}>Hello {name}!</h2>`: An `<h2>` heading that says “Hello [name]!” where `[name]` is replaced by the value of the `name` prop. We also set an inline CSS style on this `<h2>`: `style={{ color: color }}`. This is how you add CSS styles directly in JSX. We provide a JavaScript object (`{ color: color }`) where the key is a CSS property (`color`) and the value is the JavaScript variable `color` (from our props). If the `color` prop is `"blue"`, this becomes `style="{ color: 'blue' }"`, making the text blue. Inline styles let us dynamically change styling via props.
     * `<p>You are {age} years old.</p>`: A paragraph that says “You are [age] years old.”, inserting the `age` prop. If age is 15, it would read “You are 15 years old.”
   * `export default Greeting;`: This exports the component so that other files (like our `App.jsx`) can import and use it.

   In summary, `Greeting.jsx` defines a component that takes in a `name`, an `age`, and a `color`, and displays a personalized greeting with those values. We’ve used destructuring to access props and used an inline style to color the greeting text.

### B.2: Use the Greeting Component in App

Now that we have a `Greeting` component, we want to use it inside our main App. We’ll import it into `App.jsx` and include it in the JSX with some example props to test it out.

1. **Open `src/App.jsx` again**:

   * If it’s not still open, open it with:

     ```bash
     notepad src\App.jsx
     ```
   * We’ll now modify this file to include our new component.

2. **Import the Greeting component**:

   * At the top of `App.jsx`, below the other imports (or at least at the top of the file), add:

     ```jsx
     import Greeting from './Greeting.jsx';
     ```

     This tells our App component that there is a child component we want to use, coming from the `Greeting.jsx` file we just created.

3. **Use `<Greeting />` in the App’s return**:

   * In the `return (...);` of the App component, include our Greeting. For now, we can hard-code some props to test it. Modify the return to something like:

     ```jsx
     return (
       <div>
         <h1>Hello React World with Vite!</h1>
         <Greeting name="Alice" age={12} color="purple" />
       </div>
     );
     ```

     Here we:

     * Wrapped our existing `<h1>` in a `<div>` along with the new `<Greeting ... />`. We need the `<div>` because now we have two sibling elements (`<h1>` and `<Greeting>`), and JSX requires one parent element to wrap them (similar to how we used a single `<div>` in Greeting).
     * Added `<Greeting name="Alice" age={12} color="purple" />`. This uses the Greeting component and passes three props:

       * `name="Alice"` (this is a string prop, so we put it in quotes),
       * `age={12}` (this is a number prop; notice we use curly braces for non-string values in JSX, so 12 is passed as a number, not a string),
       * `color="purple"` (another string prop, which our Greeting will use to color the text).

   * After the change, your `App.jsx` might look like:

     ```jsx
     import React from 'react';
     import Greeting from './Greeting.jsx';

     function App() {
       return (
         <div>
           <h1>Hello React World with Vite!</h1>
           <Greeting name="Alice" age={12} color="purple" />
         </div>
       );
     }

     export default App;
     ```

   * Save the file.

4. **See it in action**:

   * Once you save, check the browser at `http://localhost:5173`. It should refresh and now show two lines:

     * “Hello React World with Vite!” (our H1)
     * “Hello Alice!” (in purple, as an H2, from our Greeting component), and just below it “You are 12 years old.” (the paragraph from Greeting).
   * Great, our App component is now rendering a child component! We passed props (name, age, color) to `<Greeting ...>` and the Greeting component rendered those values correctly.

5. **Explanation**:

   * We’ve demonstrated how one component (App) can **use** another component (Greeting). This is like composition: App is a parent that includes Greeting as a child in its JSX.
   * By providing different prop values, we could reuse `<Greeting />` to show different messages. Try changing the `name`, `age`, or `color` props in the `<Greeting ... />` usage in `App.jsx` (for example, change to `name="Bob"` or `color="green"`), save, and see the update in the browser. The Greeting component stays the same, but it displays different content based on props – that’s reusability!
   * We also used an inline style via props, which is a simple way to demonstrate how styling can be dynamic. In larger projects, you might use CSS files or other styling techniques, but knowing how to use the `style` prop is useful (especially for dynamic styles coming from data).
   * Destructuring props in the function signature (`function Greeting({ name, age, color })`) is a common and clean pattern in React, making the code more readable.

Now that we have a component capable of displaying a greeting, let’s make our app interactive by allowing the user to input their name and age, and see the greeting (and other messages) update automatically.

## Step 5: Exercise C – Adding Interactivity with State and Events

In this exercise, we’ll turn our static greeting into an interactive form. We’ll use **state** (via `useState`) to store the user’s input (name and age), and update that state with **onChange** events as the user types. We’ll also enhance the Greeting component to display different messages based on the state (like whether the user can vote, and a train fare based on age). This will show conditional rendering using the **ternary operator** in JSX.

### C.1: Add State for Name and Age in App Component

Our App component will manage the form’s data, so we need state variables for the name and age.

1. **Open `src/App.jsx`** (if not already open).

2. **Import useState**:

   * At the top of the file, make sure we have:

     ```jsx
     import React, { useState } from 'react';
     ```

     If you removed the `useState` import earlier, add it back. We need to import the `useState` hook from React.

3. **Initialize state variables**:

   * Inside the `App` function (at the top, before the `return` statement), add:

     ```jsx
     const [name, setName] = useState('');
     const [age, setAge] = useState(0);
     ```

     * This creates a state variable `name` (initialized to an empty string `''`) with a setter function `setName`. We’ll use `name` to store whatever the user types as their name.
     * It also creates a state variable `age` (initialized to `0`) with a setter `setAge`. We’ll use `age` to store the user’s age. (We start at 0; we could also start with `''` empty string. Using 0 is fine for now, meaning if the user hasn’t entered an age, we treat it as 0.)
     * Remember: calling these setters (`setName` or `setAge`) will update the state and cause the component to re-render.

4. **Add input fields in JSX**:

   * We will add two input elements to our App’s return JSX so the user can type their name and age. Modify the JSX to include labels and inputs, for example:

     ```jsx
     return (
       <div>
         <h1>Hello React World with Vite!</h1>

         <p>
           Name: <input 
                    type="text" 
                    value={name} 
                    onChange={e => setName(e.target.value)} 
                 />
         </p>
         <p>
           Age: <input 
                   type="number" 
                   value={age} 
                   onChange={e => setAge(Number(e.target.value))} 
                />
         </p>

         <Greeting name={name} age={age} color="purple" />
       </div>
     );
     ```

     Let’s break down what we added:

     * We wrapped each input in a `<p>` tag with a label (“Name:” and “Age:”) to keep them on separate lines and labeled.
     * **Name input**: `<input type="text" ... />` is a regular text box.

       * We set its `value` attribute to the React state variable `{name}`. This makes the input a *controlled component*, meaning its displayed value is tied to React state.
       * We add an `onChange` handler: `onChange={e => setName(e.target.value)}`. This means whenever the user types or changes the text, this arrow function runs. It takes the event `e`, and does `setName(e.target.value)`. `e.target.value` is the new text in the input. So we update the `name` state to that new text. As a result, React re-renders App (and child components) with the new `name` value.
     * **Age input**: `<input type="number" ... />` creates a number input (with up/down arrows in some browsers).

       * We bind its `value` to the `age` state.
       * `onChange={e => setAge(Number(e.target.value))}`: Here we take the input value and wrap it with `Number(...)` to convert the string input to a number type (since even number inputs give a string value by default). We then update the `age` state. Now `age` state will always hold a number. As the user types or uses the arrows, `age` updates.
     * We changed the `Greeting` usage to pass `name={name}` and `age={age}` instead of the hard-coded "Alice" and 12. We left `color="purple"` as is (so the greeting text will remain purple). Now the Greeting will receive whatever the current state values are.
   * Save your changes to `App.jsx`.

5. **View and test the interactive inputs**:

   * Look at the browser at `http://localhost:5173`. You should now see input boxes under the H1. Try typing your name into the Name field and a number into the Age field.
   * As you type your name, you should see the greeting (below the inputs, from the `<Greeting>` component) update in real-time to say “Hello [Your Name]!”.
   * As you change the age number, the text “You are [age] years old.” will update instantly as well.
   * This is React in action: each keystroke triggers `setName` or `setAge`, updating state, and React re-renders the Greeting component with the new props. It feels instantaneous.

6. **Full updated App.jsx**:

   * After adding the state and the input elements, your `App.jsx` file should look like this:

     ```jsx
     import React, { useState } from 'react';
     import Greeting from './Greeting.jsx';

     function App() {
       const [name, setName] = useState('');
       const [age, setAge] = useState(0);

       return (
         <div>
           <h1>Hello React World with Vite!</h1>

           <p>
             Name: <input
                     type="text"
                     value={name}
                     onChange={e => setName(e.target.value)}
                   />
           </p>
           <p>
             Age: <input
                    type="number"
                    value={age}
                    onChange={e => setAge(Number(e.target.value))}
                  />
           </p>

           <Greeting name={name} age={age} color="purple" />
         </div>
       );
     }

     export default App;
     ```

   * Notice how the state (`name` and `age`) is now fully integrated with the inputs and the Greeting component.

This makes our greeting dynamic! But we’re not done yet. Next, we’ll add a conditional message about voting and calculate a train fare based on age, to demonstrate conditional rendering with a ternary operator.

### C.2: Add Conditional Rendering (Voting Message and Train Fare)

We want our app to display messages that depend on the user’s age:

* A **voting eligibility message**: e.g., if age is 18 or above, “You can vote!”, otherwise “You cannot vote yet.”
* A **train fare**: a pretend scenario where children (under 18) pay one price and adults pay another.

We’ll implement these inside the `Greeting` component, since it already displays age-related info. We’ll use the **ternary operator** (`condition ? trueCase : falseCase`) within JSX to conditionally render different text.

1. **Open `src/Greeting.jsx`** in Notepad (if not already open).

2. **Update the Greeting return JSX**:

   * We will add two new `<p>` elements for the additional messages, and adjust existing text. Modify `Greeting.jsx` to something like:

     ```jsx
     function Greeting({ name, age, color }) {
       return (
         <div>
           <h2 style={{ color: color }}>Hello, {name}!</h2>
           <p>You are {age} years old.</p>
           <p>
             {age >= 18 ? "You can vote!" : "You cannot vote yet."}
           </p>
           <p>
             Your train fare is {age < 18 ? "£5 (child fare)" : "£10 (adult fare)"}.
           </p>
         </div>
       );
     }
     ```
   * What’s new:

     * We added a comma in “Hello, {name}!” just for natural language, not required.
     * We added a paragraph:

       ```jsx
       <p>{age >= 18 ? "You can vote!" : "You cannot vote yet."}</p>
       ```

       This uses the **ternary operator** inside JSX. It checks the condition `age >= 18`. If that condition is `true` (user is 18 or older), it returns `"You can vote!"`. If the condition is `false` (user is under 18), it returns `"You cannot vote yet."`. So the paragraph’s content will change based on the `age` prop.
     * We added another paragraph:

       ```jsx
       <p>Your train fare is {age < 18 ? "£5 (child fare)" : "£10 (adult fare)"}.</p>
       ```

       This one checks `age < 18`. If true (a child fare), it inserts "£5 (child fare)", otherwise it inserts "£10 (adult fare)". So under 18 gets £5, 18 or over gets £10 in our fictitious fare system.
     * These messages will update automatically whenever `age` changes, because React re-renders with the new `age` prop and recomputes the ternary expressions.
   * Save the changes to `Greeting.jsx`.

3. **Full updated Greeting.jsx**:

   * After adding the conditional messages, your `Greeting.jsx` component should look like this:

     ```jsx
     import React from 'react';

     function Greeting({ name, age, color }) {
       return (
         <div>
           <h2 style={{ color: color }}>Hello, {name}!</h2>
           <p>You are {age} years old.</p>
           <p>
             {age >= 18 ? "You can vote!" : "You cannot vote yet."}
           </p>
           <p>
             Your train fare is {age < 18 ? "£5 (child fare)" : "£10 (adult fare)"}.
           </p>
         </div>
       );
     }

     export default Greeting;
     ```

   * We included the import and export lines to show the complete file. The key additions are the two paragraphs with conditional expressions.

4. **View the updated app**:

   * In the browser, with your name and age inputs still present, you should now see two new lines appear below the "You are X years old." line:

     * For example, if age is 17: “You cannot vote yet.” and “Your train fare is £5 (child fare).”
     * If age is 18: it will change to “You can vote!” and “Your train fare is £10 (adult fare).”
     * If age is 65 (or anything 18 or above): still “You can vote!” and adult fare £10. (We didn’t implement a senior discount, but you can imagine extending the logic if you wanted.)
   * Try it out: type different ages and watch the messages change instantly:

     * Age 10 -> cannot vote, £5 fare.
     * Age 18 -> can vote, £10 fare.
     * Age 30 -> can vote, £10 fare.
     * Age 0 or a negative number (if your input allows) -> cannot vote, £5 fare (though negative age doesn’t make sense, our logic still treats it as under 18).
   * The name input still controls the greeting’s name. If you clear the name input, it will show “Hello, !” (with no name). If you want, you could add a condition to handle an empty name (e.g., show a generic greeting), but we’ll keep it simple.

5. **Understanding the Conditional (Ternary) Rendering**:

   * The **ternary operator** `condition ? exprIfTrue : exprIfFalse` is a concise way to choose between two values in JavaScript. In JSX, it’s very handy for outputting text or elements conditionally.
   * In our JSX, we wrapped the ternary expressions in `{ }` because they are JavaScript expressions that produce a string (or JSX). React will render the result of the expression inside the braces.
   * We could also have used an `if` statement *before* the `return` to decide on a message, but JSX cannot directly use a JavaScript `if` inside the return. Ternary is the JSX-friendly way for simple conditions.
   * Example logic: `age >= 18 ? "You can vote!" : "You cannot vote yet."`

     * If `age` is 18 or more, the expression yields `"You can vote!"`.
     * If `age` is less than 18, it yields `"You cannot vote yet."`.
   * Right now our fare logic has just two outcomes (child or adult fare). If we wanted more categories (like a senior discount), we could nest ternaries or use a separate if/else in the component code to set a variable, then use that variable in JSX. But for clarity, we kept it to a simple condition.

6. **Wrap Up Exercise C**:

   * We now have a fully interactive React app: the user provides input, and multiple parts of the UI update automatically.
   * We demonstrated **state** with `useState` (managing the form data), **events** with `onChange` (to respond to user input), and **conditional rendering** with ternary operators (for the vote and fare messages).
   * If you compare this to how you might do it with plain JavaScript: you would have to add event listeners for input changes, then manually manipulate the DOM elements’ text each time. React abstracts that away – you just change the state, and the DOM updates in the right places.
   * The code is also easier to reason about in pieces: The Greeting component focuses only on how to display the info (given name, age, etc.), and the App component manages the data and interactions. This separation of concerns makes it easier to scale up to bigger apps.
   * Conditional rendering with JSX allows us to create dynamic UIs (show or hide content, or change messages) in a very declarative way.
   * We also continued to use Vite’s dev server, which hot-reloaded our changes constantly. This tight feedback loop is great for development – you see results immediately as you code.

## Step 6: (Optional) Save Everything to GitHub

If you’ve been instructed to save your work, now is a good time to initialize a git repository and push your Week 6 project to GitHub, similar to previous weeks. This step is optional but recommended to back up your code.

1. **Initialize a Git repository** in the `WAD-week6` folder:

   ```bash
   git init
   git add .
   git commit -m "Week 6: React Interactive Form App"
   ```

   This creates a new local repository, adds all files, and makes an initial commit with a message.

2. **Create a `.gitignore` file** to avoid committing node modules (which are large and can be reinstalled from package.json) and perhaps any other unnecessary files:

   ```bash
   notepad .gitignore
   ```

   In Notepad, add the following lines:

   ```
   node_modules/
   dist/
   ```

   Save and close. This ensures the `node_modules` folder and any Vite build output (`dist`) are ignored by git.

3. **Push to GitHub** (if you have a GitHub account and want to save it remotely):

   * Go to GitHub and create a new repository called `WAD-week6` (or any name you prefer).

   * Back in Command Prompt, link your local repo to GitHub and push:

     ```bash
     git remote add origin https://github.com/<your-username>/WAD-week6.git
     git branch -M main
     git push -u origin main
     ```

   * Replace `<your-username>` with your GitHub username and ensure the URL is correct for your repo. This will upload your code to GitHub.

Now your code is safely stored in your GitHub repository (if you chose to do so). You can always refer back to it or share it with others if needed.

## Step 7: Summary and Next Steps

You’ve completed Week 6! Great job building your first React app from scratch. 🎊 Let’s recap what you accomplished and learned:

* **Set up a React project with Vite**: You created a new project folder, used Vite’s template to scaffold a React app, and ran it on a development server. You saw the default React app and successfully modified it.
* **Created React Components**: You made a `Greeting` component in its own file. You learned how to import a component into another and use it in JSX like a custom HTML tag.
* **Used Props**: You passed props (`name`, `age`, `color`) to the Greeting component. Inside `Greeting`, you used destructuring to easily work with those prop values, and displayed them in the JSX.
* **JSX and Inline Styles**: You wrote JSX code to describe UI elements. You also applied inline CSS styles using the `style={{ ... }}` attribute in JSX, driven by a prop (to color the text).
* **State and Hooks**: In the App component, you used the `useState` hook to create state variables for `name` and `age`. This allowed your app to hold onto user input values and trigger re-renders when they changed.
* **Event Handling**: You added `onChange` events to input fields to update state as the user types. This made the form inputs fully controlled by React state.
* **Conditional Rendering**: You used the JavaScript ternary operator in JSX to display different text based on a condition (age threshold for voting and fare). This is a clear example of how React can show/hide or change parts of the UI based on data.
* **Live Reloading and Development**: Throughout, you benefited from Vite’s hot module reloading to instantly see your changes. This speeds up development and makes it easier to experiment and debug.

**Next Steps**:

* Play around with your new React app! For example, try adding another input (maybe “Favorite Color”) and a state variable for it, then pass that as a prop to `Greeting` to dynamically change the text color via the `color` prop. This would reinforce what you learned about props and state.
* You can also add more conditional logic. For instance, display a different message if the name input is empty (like “Please enter your name.”) using a ternary to check if `name` is an empty string.
* In the coming weeks, you’ll likely learn how to handle more complex interactions, maybe how to fetch data from an API in React, or how to manage multiple components and routing. The fundamentals you learned here – components, props, state, and JSX – will be the foundation for all of that.
* If anything isn’t working, remember to check the browser’s console (F12) for error messages, and the terminal running Vite for hints. React errors often tell you which file and line to look at.

By mastering these basics, you’re well on your way to building richer web applications. **React** is a big library, but with what you’ve done in this lab, you have a solid understanding of its core ideas. Keep experimenting and having fun with it! Good luck, and see you next week for more web development adventures.
