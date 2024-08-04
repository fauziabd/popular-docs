# Introduction

note

If you are looking for the installation instructions, check out the [Quick Start](/docs/quick-start) section.

We will give a brief overview of what Wasp is, how it works on a high level and when to use it.

## Wasp is a tool to build modern web applications[​](#wasp-is-a-tool-to-build-modern-web-applications "Direct link to Wasp is a tool to build modern web applications")

It is an opinionated way of building **full-stack web applications**. It takes care of all three major parts of a web application: **client** (front-end), **server** (back-end) and **database**.

### Works well with your existing stack[​](#works-well-with-your-existing-stack "Direct link to Works well with your existing stack")

Wasp is not trying to do everything at once but rather focuses on the complexity that arises from connecting all the parts of the stack (client, server, database, deployment).

Wasp is using **React**, **Node.js** and **Prisma** under the hood and relies on them to define web components and server queries and actions.

### Wasp's secret sauce[​](#wasps-secret-sauce "Direct link to Wasp's secret sauce")

At the core is the Wasp compiler which takes the Wasp config and your Javascript code and outputs the client app, server app and deployment code.

![](/img/lp/wasp-compilation-diagram.png)

How the magic happens 🌈

The cool thing about having a compiler that understands your code is that it can do a lot of things for you.

Define your app in the Wasp config and get:

-   login and signup with Auth UI components,
-   full-stack type safety,
-   e-mail sending,
-   async processing jobs,
-   React Query powered data fetching,
-   security best practices,
-   and more.

You don't need to write any code for these features, Wasp will take care of it for you 🤯 And what's even better, Wasp also maintains the code for you, so you don't have to worry about keeping up with the latest security best practices. As Wasp updates, so does your app.

## So what does the code look like?[​](#so-what-does-the-code-look-like "Direct link to So what does the code look like?")

Let's say you want to build a web app that allows users to **create and share their favorite recipes**.

Let's start with the `main.wasp` file: it is the central file of your app, where you describe the app from the high level.

Let's give our app a title and let's immediately turn on the full-stack authentication via username and password:

main.wasp

```
    app RecipeApp {  title: "My Recipes",  wasp: { version: "^0.13.0" },  auth: {    methods: { usernameAndPassword: {} },    onAuthFailedRedirectTo: "/login",    userEntity: User  }}
```

Let's then add the data models for your recipes. Wasp understands and uses the models from the `schema.prisma` file. We will want to have Users and Users can own Recipes:

schema.prisma

```
    ...// Data models are defined using Prisma Schema Language.model User {  id          Int @id @default(autoincrement())  recipes     Recipe[]}model Recipe {  id          Int @id @default(autoincrement())  title       String  description String?  userId      Int  user        User @relation(fields: [userId], references: [id])}
```

Next, let's define how to do something with these data models!

We do that by defining Operations, in this case, a Query `getRecipes` and Action `addRecipe`, which are in their essence Node.js functions that execute on the server and can, thanks to Wasp, very easily be called from the client.

First, we define these Operations in our `main.wasp` file, so Wasp knows about them and can "beef them up":

main.wasp

```
    // Queries have automatic cache invalidation and are type-safe.query getRecipes {  fn: import { getRecipes } from "@src/recipe/operations",  entities: [Recipe],}// Actions are type-safe and can be used to perform side-effects.action addRecipe {  fn: import { addRecipe } from "@src/recipe/operations",  entities: [Recipe],}
```

... and then implement them in our Javascript (or TypeScript) code (we show just the query here, using TypeScript):

src/recipe/operations.ts

```
    // Wasp generates the types for you.import { type GetRecipes } from "wasp/server/operations";import { type Recipe } from "wasp/entities";export const getRecipes: GetRecipes<{}, Recipe[]> = async (_args, context) => {  return context.entities.Recipe.findMany( // Prisma query    { where: { user: { id: context.user.id } } }  );};export const addRecipe ...
```

Now we can very easily use these in our React components!

For the end, let's create a home page of our app.

First, we define it in `main.wasp`:

main.wasp

```
    ...route HomeRoute { path: "/", to: HomePage }page HomePage {  component: import { HomePage } from "@src/pages/HomePage",  authRequired: true // Will send user to /login if not authenticated.}
```

and then implement it as a React component in JS/TS (that calls the Operations we previously defined):

src/pages/HomePage.tsx

```
    import { useQuery, getRecipes } from "wasp/client/operations";import { type User } from "wasp/entities";export function HomePage({ user }: { user: User }) {  // Due to full-stack type safety, `recipes` will be of type `Recipe[]` here.  const { data: recipes, isLoading } = useQuery(getRecipes); // Calling our query here!  if (isLoading) {    return <div>Loading...</div>;  }  return (    <div>      <h1>Recipes</h1>      <ul>        {recipes ? recipes.map((recipe) => (          <li key={recipe.id}>            <div>{recipe.title}</div>            <div>{recipe.description}</div>          </li>        )) : 'No recipes defined yet!'}      </ul>    </div>  );}
```

And voila! We are listing all the recipes in our app 🎉

This was just a quick example to give you a taste of what Wasp is. For step by step tour through the most important Wasp features, check out the [Todo app tutorial](/docs/tutorial/create).

note

Above we skipped defining `/login` and `/signup` pages to keep the example a bit shorter, but those are very simple to do by using Wasp's Auth UI feature.

## When to use Wasp[​](#when-to-use-wasp "Direct link to When to use Wasp")

Wasp addresses the same core problems that typical web app frameworks are addressing, and it in big part [looks, swims and quacks](https://en.wikipedia.org/wiki/Duck_test) like a web app framework.

### Best used for[​](#best-used-for "Direct link to Best used for")

-   building full-stack web apps (like e.g. Airbnb or Asana)
-   quickly starting a web app with industry best practices
-   to be used alongside modern web dev stack (React and Node.js are currently supported)

### Avoid using Wasp for[​](#avoid-using-wasp-for "Direct link to Avoid using Wasp for")

-   building static/presentational websites
-   to be used as a no-code solution
-   to be a solve-it-all tool in a single language

## Wasp is a DSL[​](#wasp-is-a-dsl "Direct link to Wasp is a DSL")

note

You don't need to know what a DSL is to use Wasp, but if you are curious, you can read more about it below.

Wasp does not match typical expectations of a web app framework: it is not a set of libraries, it is instead a simple programming language that understands your code and can do a lot of things for you.

Wasp is a programming language, but a specific kind: it is specialized for a single purpose: **building modern web applications**. We call such languages *DSL*s (Domain Specific Language).

Other examples of *DSL*s that are often used today are e.g. *SQL* for databases and *HTML* for web page layouts. The main advantage and reason why *DSL*s exist is that they need to do only one task (e.g. database queries) so they can do it well and provide the best possible experience for the developer.

The same idea stands behind Wasp - a language that will allow developers to **build modern web applications with 10x less code and less stack-specific knowledge**.

---

# Quick Start

## Installation[​](#installation "Direct link to Installation")

Welcome, new Waspeteer 🐝!

Let's create and run our first Wasp app in 3 short steps:

1.  **To install Wasp on Linux / OSX / WSL (Windows), open your terminal and run:**
    
    ```
        curl -sSL https://get.wasp-lang.dev/installer.sh | sh
    ```
    
    ℹ️ Wasp requires Node.js and will warn you if it is missing: check below for [more details](#requirements).
    
2.  **Then, create a new app by running:**
    
    ```
        wasp new
    ```
    
3.  **Finally, run the app:**
    
    ```
        cd <my-project-name>wasp start
    ```
    

That's it 🎉 You have successfully created and served a new full-stack web app at [http://localhost:3000](http://localhost:3000) and Wasp is serving both frontend and backend for you.

Something Unclear?

Check [More Details](#more-details) section below if anything went wrong with the installation, or if you have additional questions.

Want an even faster start?

Try out [Wasp AI](/docs/wasp-ai/creating-new-app) 🤖 to generate a new Wasp app in minutes just from a title and short description!

Try Wasp Without Installing 🤔?

Give Wasp a spin in the browser with GitHub Codespaces by following the intructions in our [Tutorial App README](https://github.com/wasp-lang/wasp/tree/release/examples/tutorials/TodoApp)

### What next?[​](#what-next "Direct link to What next?")

-    👉 **Check out the [Todo App tutorial](/docs/tutorial/create), which will take you through all the core features of Wasp!** 👈
-    [Setup your editor](/docs/editor-setup) for working with Wasp.
-    Join us on [Discord](https://discord.gg/rzdnErX)! Any feedback or questions you have, we are there for you.
-    Follow Wasp development by subscribing to our newsletter: [https://wasp-lang.dev/#signup](https://wasp-lang.dev/#signup) . We usually send 1 per month, and [Matija](https://github.com/matijaSos) does his best to unleash his creativity to make them engaging and fun to read :D!

---

## More details[​](#more-details "Direct link to More details")

### Requirements[​](#requirements "Direct link to Requirements")

You must have Node.js (and NPM) installed on your machine and available in `PATH`. A version of Node.js must be >= 18.

If you need it, we recommend using [nvm](https://github.com/nvm-sh/nvm) for managing your Node.js installation version(s).

A quick guide on installing/using nvm

Install nvm via your OS package manager (`apt`, `pacman`, `homebrew`, ...) or via the [nvm](https://github.com/nvm-sh/nvm#install--update-script) install script.

Then, install a version of Node.js that you need:

```
    nvm install 20
```

Finally, whenever you need to ensure a specific version of Node.js is used, run:

```
    nvm use 20
```

to set the Node.js version for the current shell session.

You can run

```
    node -v
```

to check the version of Node.js currently being used in this shell session.

Check NVM repo for more details: [https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm).

### Installation[​](#installation-1 "Direct link to Installation")

-   Linux / macOS
-   Windows
-   From source

Open your terminal and run:

```
    curl -sSL https://get.wasp-lang.dev/installer.sh | sh
```

Running Wasp on Mac with Mx chip (arm64)

**Experiencing the 'Bad CPU type in executable' issue on a device with arm64 (Apple Silicon)?** Given that the wasp binary is built for x86 and not for arm64 (Apple Silicon), you'll need to install [Rosetta on your Mac](https://support.apple.com/en-us/HT211861) if you are using a Mac with Mx (M1, M2, ...). Rosetta is a translation process that enables users to run applications designed for x86 on arm64 (Apple Silicon). To install Rosetta, run the following command in your terminal

```
    softwareupdate --install-rosetta
```

Once Rosetta is installed, you should be able to run Wasp without any issues.

With Wasp for Windows, we are almost there: Wasp is successfully compiling and running on Windows but there is a bug or two stopping it from fully working. Check it out [here](https://github.com/wasp-lang/wasp/issues/48) if you are interested in helping.

In the meantime, the best way to start using Wasp on Windows is by using [WSL](https://learn.microsoft.com/en-us/windows/wsl/install). Once you set up Ubuntu on WSL, just follow Linux instructions for installing Wasp. You can refer to this [article](https://wasp-lang.dev/blog/2023/11/21/guide-windows-development-wasp-wsl) if you prefer a step by step guide to using Wasp in WSL environment. If you need further help, reach out to us on [Discord](https://discord.gg/rzdnErX) - we have some community members using WSL that might be able to help you.

caution

If you are using WSL2, make sure that your Wasp project is not on the Windows file system, but instead on the Linux file system. Otherwise, Wasp won't be able to detect file changes, due to the [issue in WSL2](https://github.com/microsoft/WSL/issues/4739).

If the installer is not working for you or your OS is not supported, you can try building Wasp from the source.

To install from source, you need to clone the [wasp repo](https://github.com/wasp-lang/wasp), install [Cabal](https://cabal.readthedocs.io/en/stable/getting-started.html) on your machine and then run `cabal install` from the `waspc/` dir.

If you have never built Wasp before, this might take some time due to `cabal` downloading dependencies for the first time.

Check [waspc/](https://github.com/wasp-lang/wasp/tree/main/waspc) for more details on building Wasp from the source.

---

# Editor Setup

note

This page assumes you have already installed Wasp. If you do not have Wasp installed yet, check out the [Quick Start](/docs/quick-start) guide.

Wasp comes with the Wasp language server, which gives supported editors powerful support and integration with the language.

## VSCode[​](#vscode "Direct link to VSCode")

Currently, Wasp only supports integration with VSCode. Install the [Wasp language extension](https://marketplace.visualstudio.com/items?itemName=wasp-lang.wasp) to get syntax highlighting and integration with the Wasp language server.

The extension enables:

-   syntax highlighting for `.wasp` files
-   the Prisma extension for `.prisma` files
-   scaffolding of new project files
-   code completion
-   diagnostics (errors and warnings)
-   go to definition

and more!

LSP Problems

If you are using TypeScript, your editor may sometimes report type and import errors even while `wasp start` is running.

This happens when the TypeScript Language Server gets out of sync with the current code. If you're using VS Code, you can manually restart the language server by opening the command palette and selecting *"TypeScript: Restart TS Server."* Open the command pallete with:

-   `Ctrl` + `Shift` + `P` if you're on Windows or Linux.
-   `Cmd` + `Shift` + `P` if you're on a Mac.

---

# 1\. Creating a New Project

info

You'll need to have the latest version of Wasp installed locally to follow this tutorial. If you haven't installed it yet, check out the [QuickStart](/docs/quick-start) guide!

In this section, we'll guide you through the process of creating a simple Todo app with Wasp. In the process, we'll take you through the most important and useful features of Wasp.

![How Todo App will work once it is done](/img/todo-app-tutorial-intro.gif)  
  

If you get stuck at any point (or just want to chat), reach out to us on [Discord](https://discord.gg/rzdnErX) and we will help you!

You can find the complete code of the app we're about to build [here](https://github.com/wasp-lang/wasp/tree/release/examples/tutorials/TodoApp).

## Creating a Project[​](#creating-a-project "Direct link to Creating a Project")

To setup a new Wasp project, run the following command in your terminal

```
    $ wasp new TodoApp
```

Enter the newly created directory and start the development server:

```
    $ cd TodoApp$ wasp start
```

`wasp start` will take a bit of time to start the server the first time you run it in a new project.

You will see log messages from the client, server, and database setting themselves up. When everything is ready, a new tab should open in your browser at `http://localhost:3000` with a simple placeholder page:

![Screenshot of new Wasp app](/img/wasp-new-screenshot.png)  
  

Wasp has generated for you the full front-end and back-end code of the app! Next, we'll take a closer look at how the project is structured.

## A note on supported languages[​](#a-note-on-supported-languages "Direct link to A note on supported languages")

Wasp supports both JavaScript and TypeScript out of the box, but you are free to choose between or mix JavaScript and TypeScript as you see fit.

We'll provide you with both JavaScript and TypeScript code in this tutorial. Code blocks will have a toggle to switch between vanilla JavaScript and TypeScript.

Try it out:

-   JavaScript
-   TypeScript

Welcome to JavaScript!

You are now reading the JavaScript version of the docs. The site will remember your preference as you switch pages.

You'll have a chance to change the language on every code snippet - both the snippets and the text will update accordingly.

Welcome to TypeScript!

You are now reading the TypeScript version of the docs. The site will remember your preference as you switch pages.

You'll have a chance to change the language on every code snippet - both the snippets and the text will update accordingly.

---

# 2\. Project Structure

-   JavaScript
-   TypeScript

After creating a new Wasp project, you'll get a file structure that looks like this:

```
    .├── .gitignore├── main.wasp          # Your Wasp code goes here.├── schema.prisma      # Your Prisma schema goes here.├── package.json       # Your dependencies and project info go here.├── public             # Your static files (e.g., images, favicon) go here.├── src                # Your source code (TS/JS/CSS/HTML) goes here.│   ├── Main.css│   ├── MainPage.jsx│   ├── vite-env.d.ts│   └── waspLogo.png├── tsconfig.json├── vite.config.ts├── .waspignore└── .wasproot
```

The default project uses JavaScript. To use TypeScript, you must manually rename the file `src/MainPage.jsx` to `src/MainPage.tsx`. Restart `wasp start` after you do this.

No updates to the `main.wasp` file are necessary - it stays the same regardless of the language you use.

After creating a new Wasp project and renaming the `src/MainPage.jsx` file, your project should look like this:

```
    .├── .gitignore├── main.wasp          # Your Wasp code goes here.├── schema.prisma      # Your Prisma schema goes here.├── package.json       # Your dependencies and project info go here.├── public             # Your static files (e.g., images, favicon) go here.├── src                # Your source code (TS/JS/CSS/HTML) goes here.│   ├── Main.css│   ├── MainPage.tsx   # Renamed from MainPage.jsx│   ├── vite-env.d.ts│   └── waspLogo.png├── tsconfig.json├── vite.config.ts├── .waspignore└── .wasproot
```

By *your code*, we mean the *"the code you write"*, as opposed to the code generated by Wasp. Wasp allows you to organize and structure your code however you think is best - there's no need to separate client files and server files into different directories.

We'd normally recommend organizing code by features (i.e., vertically). However, since this tutorial contains only a handful of files, there's no need for fancy organization. We'll keep it simple by placing everything in the root `src` directory.

Many other files (e.g., `tsconfig.json`, `vite-env.d.ts`, `.wasproot`, etc.) help Wasp and the IDE improve your development experience with autocompletion, IntelliSense, and error reporting.

The `vite.config.ts` file is used to configure [Vite](https://vitejs.dev/guide/), Wasp's build tool of choice. We won't be configuring Vite in this tutorial, so you can safely ignore the file. Still, if you ever end up wanting more control over Vite, you'll find everything you need to know in [custom Vite config docs](/docs/project/custom-vite-config).

The `schema.prisma` file is where you define your database schema using [Prisma](https://www.prisma.io/). We'll cover this a bit later in the tutorial.

The most important file in the project is `main.wasp`. Wasp uses the configuration within it to perform its magic. Based on what you write, it generates a bunch of code for your database, server-client communication, React routing, and more.

Let's take a closer look at `main.wasp`

## `main.wasp`[​](#mainwasp "Direct link to mainwasp")

`main.wasp` is your app's definition file. It defines the app's central components and helps Wasp to do a lot of the legwork for you.

The file is a list of *declarations*. Each declaration defines a part of your app.

The default `main.wasp` file generated with `wasp new` on the previous page looks like this:

-   JavaScript
-   TypeScript

main.wasp

```
    app TodoApp {  wasp: {    version: "^0.14.0" // Pins the version of Wasp to use.  },  title: "TodoApp" // Used as the browser tab title. Note that all strings in Wasp are double quoted!}route RootRoute { path: "/", to: MainPage }page MainPage {  // We specify that the React implementation of the page is exported from  // `src/MainPage.jsx`. This statement uses standard JS import syntax.  // Use `@src` to reference files inside the `src` folder.  component: import { MainPage } from "@src/MainPage"}
```

main.wasp

```
    app TodoApp {  wasp: {    version: "^0.14.0" // Pins the version of Wasp to use.  },  title: "TodoApp" // Used as the browser tab title. Note that all strings in Wasp are double quoted!}route RootRoute { path: "/", to: MainPage }page MainPage {  // We specify that the React implementation of the page is exported from  // `src/MainPage.tsx`. This statement uses standard JS import syntax.  // Use `@src` to reference files inside the `src` folder.  component: import { MainPage } from "@src/MainPage"}
```

This file uses three declaration types:

-   **app**: Top-level configuration information about your app.
    
-   **route**: Describes which path each page should be accessible from.
    
-   **page**: Defines a web page and the React component that gets rendered when the page is loaded.
    

In the next section, we'll explore how **route** and **page** work together to build your web app.

---

# 3\. Pages & Routes

In the default `main.wasp` file created by `wasp new`, there is a **page** and a **route** declaration:

-   JavaScript
-   TypeScript

main.wasp

```
    route RootRoute { path: "/", to: MainPage }page MainPage {  // We specify that the React implementation of the page is exported from   // `src/MainPage.jsx`. This statement uses standard JS import syntax.  // Use `@src` to reference files inside the `src` folder.  component: import { MainPage } from "@src/MainPage"}
```

main.wasp

```
    route RootRoute { path: "/", to: MainPage }page MainPage {  // We specify that the React implementation of the page is exported from  // `src/MainPage.tsx`. This statement uses standard JS import syntax.  // Use `@src` to reference files inside the `src` folder.  component: import { MainPage } from "@src/MainPage"}
```

Together, these declarations tell Wasp that when a user navigates to `/`, it should render the named export from `src/MainPage.tsx`.

## The MainPage Component[​](#the-mainpage-component "Direct link to The MainPage Component")

Let's take a look at the React component referenced by the page declaration:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import waspLogo from './waspLogo.png'import './Main.css'export const MainPage = () => {  // ...}
```

src/MainPage.tsx

```
    import waspLogo from './waspLogo.png'import './Main.css'export const MainPage = () => {  // ...}
```

This is a regular functional React component. It also uses the CSS file and a logo image that sit next to it in the `src` folder.

That is all the code you need! Wasp takes care of everything else necessary to define, build, and run the web app.

Keep Wasp start running

`wasp start` automatically picks up the changes you make, regenerates the code, and restarts the app. So keep it running in the background.

It also improves your experience by tracking the working directory and ensuring the generated code/types are up to date with your changes.

## Adding a Second Page[​](#adding-a-second-page "Direct link to Adding a Second Page")

To add more pages, you can create another set of **page** and **route** declarations. You can even add parameters to the URL path, using the same syntax as [React Router](https://reactrouter.com/web/). Let's test this out by adding a new page:

-   JavaScript
-   TypeScript

main.wasp

```
    route HelloRoute { path: "/hello/:name", to: HelloPage }page HelloPage {  component: import { HelloPage } from "@src/HelloPage"}
```

main.wasp

```
    route HelloRoute { path: "/hello/:name", to: HelloPage }page HelloPage {  component: import { HelloPage } from "@src/HelloPage"}
```

When a user visits `/hello/their-name`, Wasp will render the component exported from `src/HelloPage.tsx` and pass the URL parameter the same way as in React Router:

-   JavaScript
-   TypeScript

src/HelloPage.jsx

```
    export const HelloPage = (props) =>  {  return <div>Here's {props.match.params.name}!</div>}
```

src/HelloPage.tsx

```
    import { RouteComponentProps } from 'react-router-dom'export const HelloPage = (  props: RouteComponentProps<{ name: string }>) => {  return <div>Here's {props.match.params.name}!</div>}
```

Now you can visit `/hello/johnny` and see "Here's johnny!"

## Cleaning Up[​](#cleaning-up "Direct link to Cleaning Up")

Now that you've seen how Wasp deals with Routes and Pages, it's finally time to build the Todo app.

Start by cleaning up the starter project and removing unnecessary code and files.

First, remove most of the code from the `MainPage` component:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    export const MainPage = () => {  return <div>Hello world!</div>}
```

src/MainPage.tsx

```
    export const MainPage = () => {  return <div>Hello world!</div>}
```

At this point, the main page should look like this:

![Todo App - Hello World](/img/todo-app-hello-world.png)

You can now delete redundant files: `src/Main.css`, `src/waspLogo.png`, and `src/HelloPage.tsx` (we won't need this page for the rest of the tutorial).

Since `src/HelloPage.tsx` no longer exists, remove its `route` and `page` declarations from the `main.wasp` file.

Your Wasp file should now look like this:

-   JavaScript
-   TypeScript

main.wasp

```
    app TodoApp {  wasp: {    version: "^0.13.0"  },  title: "TodoApp"}route RootRoute { path: "/", to: MainPage }page MainPage {  component: import { MainPage } from "@src/MainPage"}
```

main.wasp

```
    app TodoApp {  wasp: {    version: "^0.13.0"  },  title: "TodoApp"}route RootRoute { path: "/", to: MainPage }page MainPage {  component: import { MainPage } from "@src/MainPage"}
```

Excellent work!

You now have a basic understanding of Wasp and are ready to start building your TodoApp. We'll implement the app's core features in the following sections.

---

# 4\. Database Entities

Entities are one of the most important concepts in Wasp and are how you define what gets stored in the database.

Wasp uses Prisma to talk to the database, and you define Entities by defining Prisma models in the `schema.prisma` file.

Since our Todo app is all about tasks, we'll define a Task entity by adding a Task model in the `schema.prisma` file:

schema.prisma

```
    // ...model Task {    id          Int     @id @default(autoincrement())    description String    isDone      Boolean @default(false)}
```

note

Read more about how Wasp Entities work in the [Entities](/docs/data-model/entities) section or how Wasp uses the `schema.prisma` file in the [Prisma Schema File](/docs/data-model/prisma-file) section.

To update the database schema to include this entity, stop the `wasp start` process, if it's running, and run:

```
    wasp db migrate-dev
```

You'll need to do this any time you change an entity's definition. It instructs Prisma to create a new database migration and apply it to the database.

To take a look at the database and the new `Task` entity, run:

```
    wasp db studio
```

This will open a new page in your browser to view and edit the data in your database.

![Todo App - Db studio showing Task schema](/img/todo-app-db-studio-task-entity.png)

Click on the `Task` entity and check out its fields! We don't have any data in our database yet, but we are about to change that.

---

# 5\. Querying the Database

We want to know which tasks we need to do, so let's list them!

The primary way of working with Entities in Wasp is with [Queries and Actions](/docs/data-model/operations/overview), collectively known as ***Operations***.

Queries are used to read an entity, while Actions are used to create, modify, and delete entities. Since we want to list the tasks, we'll want to use a Query.

To list the tasks, you must:

1.  Create a Query that fetches the tasks from the database.
2.  Update the `MainPage.tsx` to use that Query and display the results.

## Defining the Query[​](#defining-the-query "Direct link to Defining the Query")

We'll create a new Query called `getTasks`. We'll need to declare the Query in the Wasp file and write its implementation in .

### Declaring a Query[​](#declaring-a-query "Direct link to Declaring a Query")

We need to add a **query** declaration to `main.wasp` so that Wasp knows it exists:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...query getTasks {  // Specifies where the implementation for the query function is.  // The path `@src/queries` resolves to `src/queries.js`.  // No need to specify an extension.  fn: import { getTasks } from "@src/queries",  // Tell Wasp that this query reads from the `Task` entity. Wasp will  // automatically update the results of this query when tasks are modified.  entities: [Task]}
```

main.wasp

```
    // ...query getTasks {  // Specifies where the implementation for the query function is.  // The path `@src/queries` resolves to `src/queries.ts`.  // No need to specify an extension.  fn: import { getTasks } from "@src/queries",  // Tell Wasp that this query reads from the `Task` entity. Wasp will  // automatically update the results of this query when tasks are modified.  entities: [Task]}
```

### Implementing a Query[​](#implementing-a-query "Direct link to Implementing a Query")

-   JavaScript
-   TypeScript

src/queries.js

```
    export const getTasks = async (args, context) => {  return context.entities.Task.findMany({    orderBy: { id: 'asc' },  })}
```

src/queries.ts

```
    import { Task } from 'wasp/entities'import { type GetTasks } from 'wasp/server/operations'export const getTasks: GetTasks<void, Task[]> = async (args, context) => {  return context.entities.Task.findMany({    orderBy: { id: 'asc' },  })}
```

Wasp automatically generates the types `GetTasks` and `Task` based on the contents of `main.wasp`:

-   `Task` is a type corresponding to the `Task` entity you defined in `schema.prisma`.
-   `GetTasks` is a generic type Wasp automatically generated based on the `getTasks` Query you defined in `main.wasp`.

You can use these types to specify the Query's input and output types. This Query doesn't expect any arguments (its input type is `void`), but it does return an array of tasks (its output type is `Task[]`).

Annotating the Queries is optional, but highly recommended because doing so enables **full-stack type safety**. We'll see what this means in the next step.

Query function parameters:

-   `args: object`
    
    The arguments the caller passes to the Query.
    
-   `context`
    
    An object with extra information injected by Wasp. Its type depends on the Query declaration.
    

Since the Query declaration in `main.wasp` says that the `getTasks` Query uses `Task` entity, Wasp injected a [Prisma client](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/crud) for the `Task` entity as `context.entities.Task` - we used it above to fetch all the tasks from the database.

info

Queries and Actions are NodeJS functions executed on the server.

## Invoking the Query On the Frontend[​](#invoking-the-query-on-the-frontend "Direct link to Invoking the Query On the Frontend")

While we implement Queries on the server, Wasp generates client-side functions that automatically take care of serialization, network calls, and cache invalidation, allowing you to call the server code like it's a regular function.

This makes it easy for us to use the `getTasks` Query we just created in our React component:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import { getTasks, useQuery } from 'wasp/client/operations'export const MainPage = () => {  const { data: tasks, isLoading, error } = useQuery(getTasks)  return (    <div>      {tasks && <TasksList tasks={tasks} />}      {isLoading && 'Loading...'}      {error && 'Error: ' + error}    </div>  )}const TaskView = ({ task }) => {  return (    <div>      <input type="checkbox" id={String(task.id)} checked={task.isDone} />      {task.description}    </div>  )}const TasksList = ({ tasks }) => {  if (!tasks?.length) return <div>No tasks</div>  return (    <div>      {tasks.map((task, idx) => (        <TaskView task={task} key={idx} />      ))}    </div>  )}
```

src/MainPage.tsx

```
    import { Task } from 'wasp/entities'import { getTasks, useQuery } from 'wasp/client/operations'export const MainPage = () => {  const { data: tasks, isLoading, error } = useQuery(getTasks)  return (    <div>      {tasks && <TasksList tasks={tasks} />}      {isLoading && 'Loading...'}      {error && 'Error: ' + error}    </div>  )}const TaskView = ({ task }: { task: Task }) => {  return (    <div>      <input type="checkbox" id={String(task.id)} checked={task.isDone} />      {task.description}    </div>  )}const TasksList = ({ tasks }: { tasks: Task[] }) => {  if (!tasks?.length) return <div>No tasks</div>  return (    <div>      {tasks.map((task, idx) => (        <TaskView task={task} key={idx} />      ))}    </div>  )}
```

Most of this code is regular React, the only exception being the special `wasp` imports:

We could have called the Query directly using `getTasks()`, but the `useQuery` hook makes it reactive: React will re-render the component every time the Query changes. Remember that Wasp automatically refreshes Queries whenever the data is modified.

With these changes, you should be seeing the text "No tasks" on the screen:

![Todo App - No Tasks](/img/todo-app-no-tasks.png)

We'll create a form to add tasks in the next step 🪄

---

# 6\. Modifying Data

In the previous section, you learned about using Queries to fetch data. Let's now learn about Actions so you can add and update tasks in the database.

In this section, you will create:

1.  A Wasp Action that creates a new task.
2.  A React form that calls that Action when the user creates a task.

## Creating a New Action[​](#creating-a-new-action "Direct link to Creating a New Action")

Creating an Action is very similar to creating a Query.

### Declaring an Action[​](#declaring-an-action "Direct link to Declaring an Action")

We must first declare the Action in `main.wasp`:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...action createTask {  fn: import { createTask } from "@src/actions",  entities: [Task]}
```

main.wasp

```
    // ...action createTask {  fn: import { createTask } from "@src/actions",  entities: [Task]}
```

### Implementing an Action[​](#implementing-an-action "Direct link to Implementing an Action")

Let's now define a function for our `createTask` Action:

-   JavaScript
-   TypeScript

src/actions.js

```
    export const createTask = async (args, context) => {  return context.entities.Task.create({    data: { description: args.description },  })}
```

src/actions.ts

```
    import { Task } from 'wasp/entities'import { CreateTask } from 'wasp/server/operations'type CreateTaskPayload = Pick<Task, 'description'>export const createTask: CreateTask<CreateTaskPayload, Task> = async (  args,  context) => {  return context.entities.Task.create({    data: { description: args.description },  })}
```

Once again, we've annotated the Action with the `CreateTask` and `Task` types generated by Wasp. Just like with queries, defining the types on the implementation makes them available on the frontend, giving us **full-stack type safety**.

tip

We put the function in a new file `src/actions.ts`, but we could have put it anywhere we wanted! There are no limitations here, as long as the declaration in the Wasp file imports it correctly and the file is located within `src` directory.

## Invoking the Action on the Client[​](#invoking-the-action-on-the-client "Direct link to Invoking the Action on the Client")

Start by defining a form for creating new tasks.

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import {   createTask,   getTasks,   useQuery } from 'wasp/client/operations'// ... MainPage, TaskView, TaskList ...const NewTaskForm = () => {  const handleSubmit = async (event) => {    event.preventDefault()    try {      const target = event.target      const description = target.description.value      target.reset()      await createTask({ description })    } catch (err) {      window.alert('Error: ' + err.message)    }  }  return (    <form onSubmit={handleSubmit}>      <input name="description" type="text" defaultValue="" />      <input type="submit" value="Create task" />    </form>  )}
```

src/MainPage.tsx

```
    import { FormEvent } from 'react'import { Task } from 'wasp/entities'import {  createTask,  getTasks,  useQuery} from 'wasp/client/operations'// ... MainPage, TaskView, TaskList ...const NewTaskForm = () => {  const handleSubmit = async (event: FormEvent<HTMLFormElement>) => {    event.preventDefault()    try {      const target = event.target as HTMLFormElement      const description = target.description.value      target.reset()      await createTask({ description })    } catch (err: any) {      window.alert('Error: ' + err.message)    }  }  return (    <form onSubmit={handleSubmit}>      <input name="description" type="text" defaultValue="" />      <input type="submit" value="Create task" />    </form>  )}
```

Unlike Queries, you can call Actions directly (without wrapping them in a hook) because they don't need reactivity. The rest is just regular React code.

All that's left now is adding this form to the page component:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import {  createTask,  getTasks,  useQuery} from 'wasp/client/operations'const MainPage = () => {  const { data: tasks, isLoading, error } = useQuery(getTasks)  return (    <div>      <NewTaskForm />      {tasks && <TasksList tasks={tasks} />}      {isLoading && 'Loading...'}      {error && 'Error: ' + error}    </div>  )}// ... TaskView, TaskList, NewTaskForm ...
```

src/MainPage.tsx

```
    import { FormEvent } from 'react'import { Task } from 'wasp/entities'import {  createTask,  getTasks,  useQuery} from 'wasp/client/operations'const MainPage = () => {  const { data: tasks, isLoading, error } = useQuery(getTasks)  return (    <div>      <NewTaskForm />      {tasks && <TasksList tasks={tasks} />}      {isLoading && 'Loading...'}      {error && 'Error: ' + error}    </div>  )}// ... TaskList, TaskView, NewTaskForm ...
```

Great work!

You now have a form for creating new tasks.

Try creating a "Build a Todo App in Wasp" task and see it appear in the list below. The task is created on the server and saved in the database.

Try refreshing the page or opening it in another browser. You'll see the tasks are still there!

![Todo App - creating new task](/img/todo-app-new-task.png)  
  

Automatic Query Invalidation

When you create a new task, the list of tasks is automatically updated to display the new task, even though you haven't written any code that does that! Wasp handles these automatic updates under the hood.

When you declared the `getTasks` and `createTask` operations, you specified that they both use the `Task` entity. So when `createTask` is called, Wasp knows that the data `getTasks` fetches may have changed and automatically updates it in the background. This means that **out of the box, Wasp keeps all your queries in sync with any changes made through Actions**.

This behavior is convenient as a default but can cause poor performance in large apps. While there is no mechanism for overriding this behavior yet, it is something that we plan to include in Wasp in the future. This feature is tracked [here](https://github.com/wasp-lang/wasp/issues/63).

## A Second Action[​](#a-second-action "Direct link to A Second Action")

Our Todo app isn't finished if you can't mark a task as done.

We'll create a new Action to update a task's status and call it from React whenever a task's checkbox is toggled.

Since we've already created one task together, try to create this one yourself. It should be an Action named `updateTask` that receives the task's `id` and its `isDone` status. You can see our implementation below.

Solution

Declaring the Action in `main.wasp`:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...action updateTask {  fn: import { updateTask } from "@src/actions",  entities: [Task]}
```

main.wasp

```
    // ...action updateTask {  fn: import { updateTask } from "@src/actions",  entities: [Task]}
```

Implementing the Action on the server:

-   JavaScript
-   TypeScript

src/actions.js

```
    // ...export const updateTask = async ({ id, isDone }, context) => {  return context.entities.Task.update({    where: { id },    data: {      isDone: isDone,    },  })}
```

src/actions.ts

```
    import { CreateTask, UpdateTask } from 'wasp/server/operations'// ...type UpdateTaskPayload = Pick<Task, 'id' | 'isDone'>export const updateTask: UpdateTask<UpdateTaskPayload, Task> = async (  { id, isDone },  context) => {  return context.entities.Task.update({    where: { id },    data: {      isDone: isDone,    },  })}
```

You can now call `updateTask` from the React component:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    // ...import {  updateTask,  createTask,  getTasks,  useQuery,} from 'wasp/client/operations'// ... MainPage ...const TaskView = ({ task }) => {  const handleIsDoneChange = async (event) => {    try {      await updateTask({        id: task.id,        isDone: event.target.checked,      })    } catch (error) {      window.alert('Error while updating task: ' + error.message)    }  }  return (    <div>      <input        type="checkbox"        id={String(task.id)}        checked={task.isDone}        onChange={handleIsDoneChange}      />      {task.description}    </div>  )}// ... TaskList, NewTaskForm ...
```

src/MainPage.tsx

```
    import { FormEvent, ChangeEvent } from 'react'import { Task } from 'wasp/entities'import {  updateTask,  createTask,  getTasks,  useQuery,} from 'wasp/client/operations'// ... MainPage ...const TaskView = ({ task }: { task: Task }) => {  const handleIsDoneChange = async (event: ChangeEvent<HTMLInputElement>) => {    try {      await updateTask({        id: task.id,        isDone: event.target.checked,      })    } catch (error: any) {      window.alert('Error while updating task: ' + error.message)    }  }  return (    <div>      <input        type="checkbox"        id={String(task.id)}        checked={task.isDone}        onChange={handleIsDoneChange}      />      {task.description}    </div>  )}// ... TaskList, NewTaskForm ...
```

Awesome! You can now mark this task as done.

It's time to make one final addition to your app: supporting multiple users.

---

# 7\. Adding Authentication

Most modern apps need a way to create and authenticate users. Wasp makes this as easy as possible with its first-class auth support.

To add users to your app, you must:

-    Create a `User` Entity.
-    Tell Wasp to use the *Username and Password* authentication.
-    Add login and signup pages.
-    Update the main page to require authentication.
-    Add a relation between `User` and `Task` entities.
-    Modify your Queries and Actions so users can only see and modify their tasks.
-    Add a logout button.

## Creating a User Entity[​](#creating-a-user-entity "Direct link to Creating a User Entity")

Since Wasp manages authentication, it will create [the auth related entities](/docs/auth/entities) for you in the background. Nothing to do here!

You must only add the `User` Entity to keep track of who owns which tasks:

schema.prisma

```
    // ...model User {  id Int @id @default(autoincrement())}
```

## Adding Auth to the Project[​](#adding-auth-to-the-project "Direct link to Adding Auth to the Project")

Next, tell Wasp to use full-stack [authentication](/docs/auth/overview):

main.wasp

```
    app TodoApp {  wasp: {    version: "^0.13.0"  },  title: "TodoApp",  auth: {    // Tells Wasp which entity to use for storing users.    userEntity: User,    methods: {      // Enable username and password auth.      usernameAndPassword: {}    },    // We'll see how this is used in a bit.    onAuthFailedRedirectTo: "/login"  }}// ...
```

Don't forget to update the database schema by running:

```
    wasp db migrate-dev
```

By doing this, Wasp will create:

-   [Auth UI](/docs/auth/ui) with login and signup forms.
-   A `logout()` action.
-   A React hook `useAuth()`.
-   `context.user` for use in Queries and Actions.

info

Wasp also supports authentication using [Google](/docs/auth/social-auth/google), [GitHub](/docs/auth/social-auth/github), and [email](/docs/auth/email), with more on the way!

## Adding Login and Signup Pages[​](#adding-login-and-signup-pages "Direct link to Adding Login and Signup Pages")

Wasp creates the login and signup forms for us, but we still need to define the pages to display those forms on. We'll start by declaring the pages in the Wasp file:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { SignupPage } from "@src/SignupPage"}route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { LoginPage } from "@src/LoginPage"}
```

main.wasp

```
    // ...route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { SignupPage } from "@src/SignupPage"}route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { LoginPage } from "@src/LoginPage"}
```

Great, Wasp now knows these pages exist!

Here's the React code for the pages you've just imported:

-   JavaScript
-   TypeScript

src/LoginPage.jsx

```
    import { Link } from 'react-router-dom'import { LoginForm } from 'wasp/client/auth'export const LoginPage = () => {  return (    <div style={{ maxWidth: '400px', margin: '0 auto' }}>      <LoginForm />      <br />      <span>        I don't have an account yet (<Link to="/signup">go to signup</Link>).      </span>    </div>  )}
```

src/LoginPage.tsx

```
    import { Link } from 'react-router-dom'import { LoginForm } from 'wasp/client/auth'export const LoginPage = () => {  return (    <div style={{ maxWidth: '400px', margin: '0 auto' }}>      <LoginForm />      <br />      <span>        I don't have an account yet (<Link to="/signup">go to signup</Link>).      </span>    </div>  )}
```

The signup page is very similar to the login page:

-   JavaScript
-   TypeScript

src/SignupPage.jsx

```
    import { Link } from 'react-router-dom'import { SignupForm } from 'wasp/client/auth'export const SignupPage = () => {  return (    <div style={{ maxWidth: '400px', margin: '0 auto' }}>      <SignupForm />      <br />      <span>        I already have an account (<Link to="/login">go to login</Link>).      </span>    </div>  )}
```

src/SignupPage.tsx

```
    import { Link } from 'react-router-dom'import { SignupForm } from 'wasp/client/auth'export const SignupPage = () => {  return (    <div style={{ maxWidth: '400px', margin: '0 auto' }}>      <SignupForm />      <br />      <span>        I already have an account (<Link to="/login">go to login</Link>).      </span>    </div>  )}
```

## Update the Main Page to Require Auth[​](#update-the-main-page-to-require-auth "Direct link to Update the Main Page to Require Auth")

We don't want users who are not logged in to access the main page, because they won't be able to create any tasks. So let's make the page private by requiring the user to be logged in:

main.wasp

```
    // ...page MainPage {  authRequired: true,  component: import { MainPage } from "@src/MainPage"}
```

Now that auth is required for this page, unauthenticated users will be redirected to `/login`, as we specified with `app.auth.onAuthFailedRedirectTo`.

Additionally, when `authRequired` is `true`, the page's React component will be provided a `user` object as prop.

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    export const MainPage = ({ user }) => {  // Do something with the user  // ...}
```

src/MainPage.tsx

```
    import { AuthUser } from 'wasp/auth'export const MainPage = ({ user }: { user: AuthUser }) => {  // Do something with the user  // ...}
```

Ok, time to test this out. Navigate to the main page (`/`) of the app. You'll get redirected to `/login`, where you'll be asked to authenticate.

Since we just added users, you don't have an account yet. Go to the signup page and create one. You'll be sent back to the main page where you will now be able to see the TODO list!

Let's check out what the database looks like. Start the Prisma Studio:

```
    wasp db studio
```

![Database demonstration - password hashing](/img/wasp_user_in_db.gif)

You'll notice that we now have a `User` entity in the database alongside the `Task` entity.

However, you will notice that if you try logging in as different users and creating some tasks, all users share the same tasks. That's because you haven't yet updated the queries and actions to have per-user tasks. Let's do that next.

You might notice some extra Prisma models like `Auth`, `AuthIdentity` and `Session` that Wasp created for you. You don't need to care about these right now, but if you are curious, you can read more about them [here](/docs/auth/entities).

## Defining a User-Task Relation[​](#defining-a-user-task-relation "Direct link to Defining a User-Task Relation")

First, let's define a one-to-many relation between users and tasks (check the [Prisma docs on relations](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-schema/relations)):

schema.prisma

```
    // ...model User {  id    Int    @id @default(autoincrement())  tasks Task[]}model Task {  id          Int     @id @default(autoincrement())  description String  isDone      Boolean @default(false)  user        User?   @relation(fields: [userId], references: [id])  userId      Int?}
```

As always, you must migrate the database after changing the Entities:

```
    wasp db migrate-dev
```

note

We made `user` and `userId` in `Task` optional (via `?`) because that allows us to keep the existing tasks, which don't have a user assigned, in the database.

This isn't recommended because it allows an unwanted state in the database (what is the purpose of the task not belonging to anybody?) and normally we would not make these fields optional.

Instead, we would do a data migration to take care of those tasks, even if it means just deleting them all. However, for this tutorial, for the sake of simplicity, we will stick with this.

## Updating Operations to Check Authentication[​](#updating-operations-to-check-authentication "Direct link to Updating Operations to Check Authentication")

Next, let's update the queries and actions to forbid access to non-authenticated users and to operate only on the currently logged-in user's tasks:

-   JavaScript
-   TypeScript

src/queries.js

```
    import { HttpError } from 'wasp/server'export const getTasks = async (args, context) => {  if (!context.user) {    throw new HttpError(401)  }  return context.entities.Task.findMany({    where: { user: { id: context.user.id } },    orderBy: { id: 'asc' },  })}
```

src/queries.ts

```
    import { Task } from 'wasp/entities'import { HttpError } from 'wasp/server'import { GetTasks } from 'wasp/server/operations'export const getTasks: GetTasks<void, Task[]> = async (args, context) => {  if (!context.user) {    throw new HttpError(401)  }  return context.entities.Task.findMany({    where: { user: { id: context.user.id } },    orderBy: { id: 'asc' },  })}
```

-   JavaScript
-   TypeScript

src/actions.js

```
    import { HttpError } from 'wasp/server'export const createTask = async (args, context) => {  if (!context.user) {    throw new HttpError(401)  }  return context.entities.Task.create({    data: {      description: args.description,      user: { connect: { id: context.user.id } },    },  })}export const updateTask = async (args, context) => {  if (!context.user) {    throw new HttpError(401)  }  return context.entities.Task.updateMany({    where: { id: args.id, user: { id: context.user.id } },    data: { isDone: args.isDone },  })}
```

src/actions.ts

```
    import { Task } from 'wasp/entities'import { HttpError } from 'wasp/server'import { CreateTask, UpdateTask } from 'wasp/server/operations'type CreateTaskPayload = Pick<Task, 'description'>export const createTask: CreateTask<CreateTaskPayload, Task> = async (  args,  context) => {  if (!context.user) {    throw new HttpError(401)  }  return context.entities.Task.create({    data: {      description: args.description,      user: { connect: { id: context.user.id } },    },  })}type UpdateTaskPayload = Pick<Task, 'id' | 'isDone'>export const updateTask: UpdateTask<  UpdateTaskPayload,  { count: number }> = async ({ id, isDone }, context) => {  if (!context.user) {    throw new HttpError(401)  }  return context.entities.Task.updateMany({    where: { id, user: { id: context.user.id } },    data: { isDone },  })}
```

note

Due to how Prisma works, we had to convert `update` to `updateMany` in `updateTask` action to be able to specify the user id in `where`.

With these changes, each user should have a list of tasks that only they can see and edit.

Try playing around, adding a few users and some tasks for each of them. Then open the DB studio:

```
    wasp db studio
```

![Database demonstration](/img/wasp_db_demonstration.gif)

You will see that each user has their tasks, just as we specified in our code!

## Logout Button[​](#logout-button "Direct link to Logout Button")

Last, but not least, let's add the logout functionality:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    // ...import { logout } from 'wasp/client/auth'//...const MainPage = () => {  // ...  return (    <div>      // ...      <button onClick={logout}>Logout</button>    </div>  )}
```

src/MainPage.tsx

```
    // ...import { logout } from 'wasp/client/auth'//...const MainPage = () => {  // ...  return (    <div>      // ...      <button onClick={logout}>Logout</button>    </div>  )}
```

This is it, we have a working authentication system, and our Todo app is multi-user!

## What's Next?[​](#whats-next "Direct link to What's Next?")

We did it 🎉 You've followed along with this tutorial to create a basic Todo app with Wasp.

You should be ready to learn about more complicated features and go more in-depth with the features already covered. Scroll through the sidebar on the left side of the page to see every feature Wasp has to offer. Or, let your imagination run wild and start building your app! ✨

Looking for inspiration?

-   Get a jump start on your next project with [Starter Templates](/docs/project/starter-templates).
-   Check out our [official examples](https://github.com/wasp-lang/wasp/tree/release/examples).
-   Make a real-time app with [Web Sockets](/docs/advanced/web-sockets).

note

If you notice that some of the features you'd like to have are missing, or have any other kind of feedback, please write to us on [Discord](https://discord.gg/rzdnErX) or create an issue on [Github](https://github.com/wasp-lang/wasp), so we can learn which features to add/improve next 🙏

If you would like to contribute or help to build a feature, let us know! You can find more details on contributing [here](/docs/contributing).

Oh, and do [**subscribe to our newsletter**](/#signup)! We usually send one per month, and Matija does his best to unleash his creativity to make them engaging and fun to read :D!

---

# Entities

Entities are the foundation of your app's data model. In short, an Entity defines a model in your database.

Wasp uses the excellent [Prisma ORM](https://www.prisma.io/) to implement all database functionality and occasionally enhances it with a thin abstraction layer. This means that you use the `schema.prisma` file to define your database models and relationships. Wasp understands the Prisma schema file and picks up all the models you define there. You can read more about this in the [Prisma Schema File](/docs/data-model/prisma-file) section of the docs.

In your project, you'll find a `schema.prisma` file in the root directory:

```
    .├── main.wasp...├── schema.prisma├── src├── tsconfig.json└── vite.config.ts
```

Prisma uses the *Prisma Schema Language*, a simple definition language explicitly created for defining models. The language is declarative and very intuitive. We'll also go through an example later in the text, so there's no need to go and thoroughly learn it right away. Still, if you're curious, look no further than Prisma's official documentation:

-   [Basic intro and examples](https://www.prisma.io/docs/orm/prisma-schema/overview)
-   [A more exhaustive language specification](https://www.prisma.io/docs/orm/reference/prisma-schema-reference)

## Defining an Entity[​](#defining-an-entity "Direct link to Defining an Entity")

A Prisma `model` declaration in the `schema.prisma` file represents a Wasp Entity.

Entity vs Model

You might wonder why we distinguish between a **Wasp Entity** and a **Prisma model** if they're essentially the same thing right now.

While defining a Prisma model is currently the only way to create an Entity in Wasp, the Entity concept is a higher-level abstraction. We plan to expand on Entities in the future, both in terms of how you can define them and what you can do with them.

So, think of an Entity as a Wasp concept and a model as a Prisma concept. For now, all Prisma models are Entities and vice versa, but this relationship might evolve as Wasp grows.

Here's how you could define an Entity that represents a Task:

-   JavaScript
-   TypeScript

schema.prisma

```
    model Task {  id          String  @id @default(uuid())  description String  isDone      Boolean @default(false)}
```

schema.prisma

```
    model Task {  id          String  @id @default(uuid())  description String  isDone      Boolean @default(false)}
```

The above Prisma `model` definition tells Wasp to create a table for storing Tasks where each task has three fields (i.e., the `tasks` table has three columns):

-   `id` - A string value serving as a primary key. The database automatically generates it by generating a random unique ID.
-   `description` - A string value for storing the task's description.
-   `isDone` - A boolean value indicating the task's completion status. If you don't set it when creating a new task, the database sets it to `false` by default.

### Working with Entities[​](#working-with-entities "Direct link to Working with Entities")

Let's see how you can define and work with Wasp Entities:

1.  Create/update some Entities in the `schema.prisma` file.
2.  Run `wasp db migrate-dev`. This command syncs the database model with the Entity definitions the `schema.prisma` file. It does this by creating migration scripts.
3.  Migration scripts are automatically placed in the `migrations/` folder. Make sure to commit this folder into version control.
4.  Use Wasp's JavasScript API to work with the database when implementing Operations (we'll cover this in detail when we talk about [operations](/docs/data-model/operations/overview)).

#### Using Entities in Operations[​](#using-entities-in-operations "Direct link to Using Entities in Operations")

Most of the time, you will be working with Entities within the context of [Operations (Queries & Actions)](/docs/data-model/operations/overview). We'll see how that's done on the next page.

#### Using Entities directly[​](#using-entities-directly "Direct link to Using Entities directly")

If you need more control, you can directly interact with Entities by importing and using the [Prisma Client](https://www.prisma.io/docs/concepts/components/prisma-client/crud). We recommend sticking with conventional Wasp-provided mechanisms, only resorting to directly using the Prisma client only if you need a feature Wasp doesn't provide.

You can only use the Prisma Client in your Wasp server code. You can import it like this:

-   JavaScript
-   TypeScript

```
    import { prisma } from 'wasp/server'prisma.task.create({    description: "Read the Entities doc",    isDone: true // almost :)})
```

```
    import { prisma } from 'wasp/server'prisma.task.create({    description: "Read the Entities doc",    isDone: true // almost :)})
```

### Next steps[​](#next-steps "Direct link to Next steps")

Now that we've seen how to define Entities that represent Wasp's core data model, we'll see how to make the most of them in other parts of Wasp. Keep reading to learn all about Wasp Operations!

---

# Overview

While Entities enable you to define your app's data model and relationships, Operations are all about working with this data.

There are two kinds of Operations: [Queries](/docs/data-model/operations/queries) and [Actions](/docs/data-model/operations/actions). As their names suggest, Queries are meant for reading data, and Actions are meant for changing it (either by updating existing entries or creating new ones).

Keep reading to find out all there is to know about Operations in Wasp.

---

# Queries

We'll explain what Queries are and how to use them. If you're looking for a detailed API specification, skip ahead to the [API Reference](#api-reference).

You can use Queries to fetch data from the server. They shouldn't modify the server's state. Fetching all comments on a blog post, a list of users that liked a video, information about a single product based on its ID... All of these are perfect use cases for a Query.

tip

Queries are fairly similar to Actions in terms of their API. Therefore, if you're already familiar with Actions, you might find reading the entire guide repetitive.

We instead recommend skipping ahead and only reading [the differences between Queries and Actions](/docs/data-model/operations/actions#differences-between-queries-and-actions), and consulting the [API Reference](#api-reference) as needed.

## Working with Queries[​](#working-with-queries "Direct link to Working with Queries")

You declare queries in the `.wasp` file and implement them using NodeJS. Wasp not only runs these queries within the server's context but also creates code that enables you to call them from any part of your codebase, whether it's on the client or server side.

This means you don't have to build an HTTP API for your query, manage server-side request handling, or even deal with client-side response handling and caching. Instead, just concentrate on implementing the business logic inside your query, and let Wasp handle the rest!

To create a Query, you must:

1.  Declare the Query in Wasp using the `query` declaration.
2.  Define the Query's NodeJS implementation.

After completing these two steps, you'll be able to use the Query from any point in your code.

### Declaring Queries[​](#declaring-queries "Direct link to Declaring Queries")

To create a Query in Wasp, we begin with a `query` declaration.

Let's declare two Queries - one to fetch all tasks, and another to fetch tasks based on a filter, such as whether a task is done:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...query getAllTasks {  fn: import { getAllTasks } from "@src/queries.js"}query getFilteredTasks {  fn: import { getFilteredTasks } from "@src/queries.js"}
```

main.wasp

```
    // ...query getAllTasks {  fn: import { getAllTasks } from "@src/queries.js"}query getFilteredTasks {  fn: import { getFilteredTasks } from "@src/queries.js"}
```

If you want to know about all supported options for the `query` declaration, take a look at the [API Reference](#api-reference).

The names of Wasp Queries and their implementations don't need to match, but we'll keep them the same to avoid confusion.

info

You might have noticed that we told Wasp to import Query implementations that don't yet exist. Don't worry about that for now. We'll write the implementations imported from `queries.ts` in the next section.

It's a good idea to start with the high-level concept (the Query declaration in the Wasp file) and only then deal with the implementation details (the Query's implementation in JavaScript).

After declaring a Wasp Query, two important things happen:

-   Wasp **generates a server-side NodeJS function** that shares its name with the Query.
    
-   Wasp **generates a client-side JavaScript function** that shares its name with the Query (e.g., `getFilteredTasks`). This function takes a single optional argument - an object containing any serializable data you wish to use inside the Query. Wasp will send this object over the network and pass it into the Query's implementation as its first positional argument (more on this when we look at the implementations). Such an abstraction works thanks to an HTTP API route handler Wasp generates on the server, which calls the Query's NodeJS implementation under the hood.
    

Generating these two functions ensures a similar calling interface across the entire app (both client and server).

### Implementing Queries in Node[​](#implementing-queries-in-node "Direct link to Implementing Queries in Node")

Now that we've declared the Query, what remains is to implement it. We've instructed Wasp to look for the Queries' implementations in the file `src/queries.ts`, so that's where we should export them from.

Here's how you might implement the previously declared Queries `getAllTasks` and `getFilteredTasks`:

-   JavaScript
-   TypeScript

src/queries.js

```
    // our "database"const tasks = [  { id: 1, description: 'Buy some eggs', isDone: true },  { id: 2, description: 'Make an omelette', isDone: false },  { id: 3, description: 'Eat breakfast', isDone: false },]// You don't need to use the arguments if you don't need themexport const getAllTasks = () => {  return tasks}// The 'args' object is something sent by the caller (most often from the client)export const getFilteredTasks = (args) => {  const { isDone } = args  return tasks.filter((task) => task.isDone === isDone)}
```

Payload constraints

Wasp uses [superjson](https://github.com/blitz-js/superjson) under the hood. This means you're not limited to only sending and receiving JSON payloads.

You can send and receive any superjson-compatible payload (like Dates, Sets, Lists, circular references, etc.) and let Wasp handle the (de)serialization.

src/queries.ts

```
    import { type GetAllTasks, type GetFilteredTasks } from 'wasp/server/operations'type Task = {  id: number  description: string  isDone: boolean}// our "database"const tasks: Task[] = [  { id: 1, description: 'Buy some eggs', isDone: true },  { id: 2, description: 'Make an omelette', isDone: false },  { id: 3, description: 'Eat breakfast', isDone: false },]// You don't need to use the arguments if you don't need themexport const getAllTasks: GetAllTasks<void, Task[]> = () => {  return tasks}// The 'args' object is something sent by the caller (most often from the client)export const getFilteredTasks: GetFilteredTasks<  Pick<Task, 'isDone'>,  Task[]> = (args) => {  const { isDone } = args  return tasks.filter((task) => task.isDone === isDone)}
```

Payload constraints

Wasp uses [superjson](https://github.com/blitz-js/superjson) under the hood. This means you're not limited to only sending and receiving JSON payloads.

You can send and receive any superjson-compatible payload (like Dates, Sets, Lists, circular references, etc.) and let Wasp handle the (de)serialization.

#### Type support for Queries[​](#type-support-for-queries "Direct link to Type support for Queries")

Wasp automatically generates the types `GetAllTasks` and `GetFilteredTasks` based on your Wasp file's declarations:

-   `GetAllTasks` is a generic type automatically generated by Wasp, based on the Query declaration for `getAllTasks`.
-   `GetFilteredTasks` is also a generic type automatically generated by Wasp, based on the Query declaration for `getFilteredTasks`.

Use these types to type the Query's implementation. It's optional but very helpful since doing so properly types the Query's context.

In this case, TypeScript will know the `context.entities` object must include the `Task` entity. TypeScript also knows whether the `context` object includes user information (it depends on whether your Query uses auth).

The generated types are generic and accept two optional type arguments: `Input` and `Output`.

1.  `Input` - The argument (the payload) received by the Query function.
2.  `Output` - The Query function's return type.

Use these type arguments to type the Query's inputs and outputs.

Explanation for the example above

The above code says that the Query `getAllTasks` doesn't expect any arguments (its input type is `void`), but it does return a list of tasks (its output type is `Task[]`).

On the other hand, the Query `getFilteredTasks` expects an object of type `{ isDone: boolean }`. This type is derived from the `Task` entity type.

If you don't care about typing the Query's inputs and outputs, you can omit both type arguments. TypeScript will then infer the most general types (`never` for the input and `unknown` for the output).

Specifying `Input` or `Output` is completely optional, but we highly recommended it. Doing so gives you:

-   Type support for the arguments and the return value inside the implementation.
-   **Full-stack type safety**. We'll explore what this means when we discuss calling the Query from the client.

Read more about type support for implementing Queries in the [API Reference](#implementing-queries).

Inferring the return type

If don't want to explicitly type the Query's return value, the `satisfies` keyword tells TypeScript to infer it automatically:

```
    const getFoo = (async (_args, context) => {  const foos = await context.entities.Foo.findMany()  return {    foos,    message: 'Here are some foos!',    queriedAt: new Date(),  }}) satisfies GetFoo
```

From the snippet above, TypeScript knows:

1.  The correct type for `context`.
2.  The Query's return type is `{ foos: Foo[], message: string, queriedAt: Date }`.

If you don't need the context, you can skip specifying the Query's type (and arguments):

```
    const getFoo = () => {{ name: 'Foo', date: new Date() }}
```

For a detailed explanation of the Query definition API (more precisely, its arguments and return values), check the [API Reference](#api-reference).

### Using Queries[​](#using-queries "Direct link to Using Queries")

#### Using Queries on the client[​](#using-queries-on-the-client "Direct link to Using Queries on the client")

To call a Query on the client, you can import it from `wasp/client/operations` and call it directly.

The usage doesn't change depending on whether the Query is authenticated or not. Wasp authenticates the logged-in user in the background.

-   JavaScript
-   TypeScript

```
    import { getAllTasks, getFilteredTasks } from 'wasp/client/operations'// ...const allTasks = await getAllTasks()const doneTasks = await getFilteredTasks({ isDone: true })
```

```
    import { getAllTasks, getFilteredTasks } from 'wasp/client/operations'// TypeScript automatically infers the return values and type-checks// the payloads.const allTasks = await getAllTasks()const doneTasks = await getFilteredTasks({ isDone: true })
```

Wasp supports **automatic full-stack type safety**. You only need to specify the Query's type in its server-side definition, and the client code will automatically know its API payload types.

#### Using Queries on the server[​](#using-queries-on-the-server "Direct link to Using Queries on the server")

Calling a Query on the server is similar to calling it on the client.

Here's what you have to do differently:

-   Import Queries from `wasp/server/operations` instead of `wasp/client/operations`.
-   Make sure you pass in a context object with the user to authenticated Queries.

-   JavaScript
-   TypeScript

```
    import { getAllTasks, getFilteredTasks } from 'wasp/server/operations'const user = // Get an AuthUser object, e.g., from context.user in an operation.// ...const allTasks = await getAllTasks({ user })const doneTasks = await getFilteredTasks({ isDone: true }, { user })
```

```
    import { getAllTasks, getFilteredTasks } from 'wasp/server/operations'const user = // Get an AuthUser object, e.g., from context.user in an operation.// TypeScript automatically infers the return values and type-checks// the payloads.const allTasks = await getAllTasks({ user })const doneTasks = await getFilteredTasks({ isDone: true }, { user })
```

#### The `useQuery` hook[​](#the-usequery-hook "Direct link to the-usequery-hook")

When using Queries on the client, you can make them reactive with the `useQuery` hook. This hook comes bundled with Wasp and is a thin wrapper around the `useQuery` hook from [*react-query*](https://github.com/tannerlinsley/react-query). The only difference is that you don't need to supply the key - Wasp handles this for you automatically.

Here's an example of calling the Queries using the `useQuery` hook:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import React from 'react'import { useQuery, getAllTasks, getFilteredTasks } from 'wasp/client/operations'const MainPage = () => {  const { data: allTasks, error: error1 } = useQuery(getAllTasks)  const { data: doneTasks, error: error2 } = useQuery(getFilteredTasks, {    isDone: true,  })  if (error1 !== null || error2 !== null) {    return <div>There was an error</div>  }  return (    <div>      <h2>All Tasks</h2>      {allTasks && allTasks.length > 0        ? allTasks.map((task) => <Task key={task.id} {...task} />)        : 'No tasks'}      <h2>Finished Tasks</h2>      {doneTasks && doneTasks.length > 0        ? doneTasks.map((task) => <Task key={task.id} {...task} />)        : 'No finished tasks'}    </div>  )}const Task = ({ description, isDone }: Task) => {  return (    <div>      <p>        <strong>Description: </strong>        {description}      </p>      <p>        <strong>Is done: </strong>        {isDone ? 'Yes' : 'No'}      </p>    </div>  )}export default MainPage
```

src/MainPage.tsx

```
    import React from 'react'import { type Task } from 'wasp/entities'import { useQuery, getAllTasks, getFilteredTasks } from 'wasp/client/operations'const MainPage = () => {  // TypeScript automatically infers return values and type-checks payload types.  const { data: allTasks, error: error1 } = useQuery(getAllTasks)  const { data: doneTasks, error: error2 } = useQuery(getFilteredTasks, {    isDone: true,  })  if (error1 !== null || error2 !== null) {    return <div>There was an error</div>  }  return (    <div>      <h2>All Tasks</h2>      {allTasks && allTasks.length > 0        ? allTasks.map((task) => <Task key={task.id} {...task} />)        : 'No tasks'}      <h2>Finished Tasks</h2>      {doneTasks && doneTasks.length > 0        ? doneTasks.map((task) => <Task key={task.id} {...task} />)        : 'No finished tasks'}    </div>  )}const Task = ({ description, isDone }: Task) => {  return (    <div>      <p>        <strong>Description: </strong>        {description}      </p>      <p>        <strong>Is done: </strong>        {isDone ? 'Yes' : 'No'}      </p>    </div>  )}export default MainPage
```

Notice how you don't need to annotate the Query's return value type. Wasp automatically infers the from the Query's backend implementation. This is **full-stack type safety**: the types on the client always match the types on the server.

For a detailed specification of the `useQuery` hook, check the [API Reference](#api-reference).

### Error Handling[​](#error-handling "Direct link to Error Handling")

For security reasons, all exceptions thrown in the Query's NodeJS implementation are sent to the client as responses with the HTTP status code `500`, with all other details removed. Hiding error details by default helps against accidentally leaking possibly sensitive information over the network.

If you do want to pass additional error information to the client, you can construct and throw an appropriate `HttpError` in your implementation:

-   JavaScript
-   TypeScript

src/queries.js

```
    import { HttpError } from 'wasp/server'export const getAllTasks = async (args, context) => {  throw new HttpError(    403, // status code    "You can't do this!", // message    { foo: 'bar' } // data  )}
```

src/queries.ts

```
    import { type GetAllTasks } from 'wasp/server/operations'import { HttpError } from 'wasp/server'export const getAllTasks: GetAllTasks = async (args, context) => {  throw new HttpError(    403, // status code    "You can't do this!", // message    { foo: 'bar' } // data  )}
```

If the status code is `4xx`, the client will receive a response object with the corresponding `message` and `data` fields, and it will rethrow the error (including these fields). To prevent information leakage, the server won't forward these fields for any other HTTP status codes.

### Using Entities in Queries[​](#using-entities-in-queries "Direct link to Using Entities in Queries")

In most cases, resources used in Queries will be [Entities](/docs/data-model/entities). To use an Entity in your Query, add it to the `query` declaration in Wasp:

-   JavaScript
-   TypeScript

main.wasp

```
    query getAllTasks {  fn: import { getAllTasks } from "@src/queries.js",  entities: [Task]}query getFilteredTasks {  fn: import { getFilteredTasks } from "@src/queries.js",  entities: [Task]}
```

main.wasp

```
    query getAllTasks {  fn: import { getAllTasks } from "@src/queries.js",  entities: [Task]}query getFilteredTasks {  fn: import { getFilteredTasks } from "@src/queries.js",  entities: [Task]}
```

Wasp will inject the specified Entity into the Query's `context` argument, giving you access to the Entity's Prisma API:

-   JavaScript
-   TypeScript

src/queries.js

```
    export const getAllTasks = async (args, context) => {  return context.entities.Task.findMany({})}export const getFilteredTasks = async (args, context) => {  return context.entities.Task.findMany({    where: { isDone: args.isDone },  })}
```

src/queries.ts

```
    import { type Task } from 'wasp/entities'import { type GetAllTasks, type GetFilteredTasks } from 'wasp/server/operations'export const getAllTasks: GetAllTasks<void, Task[]> = async (args, context) => {  return context.entities.Task.findMany({})}export const getFilteredTasks: GetFilteredTasks<  Pick<Task, 'isDone'>,  Task[]> = async (args, context) => {  return context.entities.Task.findMany({    where: { isDone: args.isDone },  })}
```

Again, annotating the Queries is optional, but greatly improves **full-stack type safety**.

The object `context.entities.Task` exposes `prisma.task` from [Prisma's CRUD API](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/crud).

## API Reference[​](#api-reference "Direct link to API Reference")

### Declaring Queries[​](#declaring-queries-1 "Direct link to Declaring Queries")

The `query` declaration supports the following fields:

-   `fn: ExtImport` required
    
    The import statement of the Query's NodeJs implementation.
    
-   `entities: [Entity]`
    
    A list of entities you wish to use inside your Query. For instructions on using Entities in Queries, take a look at [the guide](#using-entities-in-queries).
    

#### Example[​](#example "Direct link to Example")

-   JavaScript
-   TypeScript

Declaring the Query:

```
    query getFoo {    fn: import { getFoo } from "@src/queries.js"    entities: [Foo]}
```

Enables you to import and use it anywhere in your code (on the server or the client):

```
    // Use it on the clientimport { getFoo } from 'wasp/client/operations'// Use it on the serverimport { getFoo } from 'wasp/server/operations'
```

On the the client, the Query expects

Declaring the Query:

```
    query getFoo {    fn: import { getFoo } from "@src/queries.js"    entities: [Foo]}
```

Enables you to import and use it anywhere in your code (on the server or the client):

```
    // Use it on the clientimport { getFoo } from 'wasp/client/operations'// Use it on the serverimport { getFoo } from 'wasp/server/operations'
```

And also creates a type you can import on the server:

```
    import { type GetFoo } from 'wasp/server/operations'
```

### Implementing Queries[​](#implementing-queries "Direct link to Implementing Queries")

The Query's implementation is a NodeJS function that takes two arguments (it can be an `async` function if you need to use the `await` keyword). Since both arguments are positional, you can name the parameters however you want, but we'll stick with `args` and `context`:

1.  `args` (type depends on the Query)
    
    An object containing the data **passed in when calling the query** (e.g., filtering conditions). Check [the usage examples](#using-queries) to see how to pass this object to the Query.
    
2.  `context` (type depends on the Query)
    
    An additional context object **passed into the Query by Wasp**. This object contains user session information, as well as information about entities. Check the [section about using entities in Queries](#using-entities-in-queries) to see how to use the entities field on the `context` object, or the [auth section](/docs/auth/overview#using-the-contextuser-object) to see how to use the `user` object.
    

#### Example[​](#example-1 "Direct link to Example")

-   JavaScript
-   TypeScript

The following Query:

```
    query getFoo {    fn: import { getFoo } from "@src/queries.js"    entities: [Foo]}
```

Expects to find a named export `getFoo` from the file `src/queries.js`

queries.js

```
    export const getFoo = (args, context) => {  // implementation}
```

The following Query:

```
    query getFoo {    fn: import { getFoo } from "@src/queries.js"    entities: [Foo]}
```

Expects to find a named export `getFoo` from the file `src/queries.js`

You can use the generated type `GetFoo` and specify the Query's inputs and outputs using its type arguments.

queries.ts

```
    import { type GetFoo } from 'wasp/server/operations'type Foo = // ...export const getFoo: GetFoo<{ id: number }, Foo> = (args, context) => {  // implementation};
```

In this case, the Query expects to receive an object with an `id` field of type `number` (this is the type of `args`), and return a value of type `Foo` (this must match the type of the Query's return value).

### The `useQuery` Hook[​](#the-usequery-hook-1 "Direct link to the-usequery-hook-1")

Wasp's `useQuery` hook is a thin wrapper around the `useQuery` hook from [*react-query*](https://github.com/tannerlinsley/react-query). One key difference is that Wasp doesn't expect you to supply the cache key - it takes care of it under the hood.

Wasp's `useQuery` hook accepts three arguments:

-   `queryFn` required
    
    The client-side query function generated by Wasp based on a `query` declaration in your `.wasp` file.
    
-   `queryFnArgs`
    
    The arguments object (payload) you wish to pass into the Query. The Query's NodeJS implementation will receive this object as its first positional argument.
    
-   `options`
    
    A *react-query* `options` object. Use this to change [the default behavior](https://react-query.tanstack.com/guides/important-defaults) for this particular Query. If you want to change the global defaults, you can do so in the [client setup function](/docs/project/client-config#overriding-default-behaviour-for-queries).
    

For an example of usage, check [this section](#the-usequery-hook).

---

# Actions

We'll explain what Actions are and how to use them. If you're looking for a detailed API specification, skip ahead to the [API Reference](#api-reference).

Actions are quite similar to [Queries](/docs/data-model/operations/queries), but with a key distinction: Actions are designed to modify and add data, while Queries are solely for reading data. Examples of Actions include adding a comment to a blog post, liking a video, or updating a product's price.

Actions and Queries work together to keep data caches up-to-date.

tip

Actions are almost identical to Queries in terms of their API. Therefore, if you're already familiar with Queries, you might find reading the entire guide repetitive.

We instead recommend skipping ahead and only reading [the differences between Queries and Actions](#differences-between-queries-and-actions), and consulting the [API Reference](#api-reference) as needed.

## Working with Actions[​](#working-with-actions "Direct link to Working with Actions")

Actions are declared in Wasp and implemented in NodeJS. Wasp runs Actions within the server's context, but it also generates code that allows you to call them from anywhere in your code (either client or server) using the same interface.

This means you don't have to worry about building an HTTP API for the Action, managing server-side request handling, or even dealing with client-side response handling and caching. Instead, just focus on developing the business logic inside your Action, and let Wasp handle the rest!

To create an Action, you need to:

1.  Declare the Action in Wasp using the `action` declaration.
2.  Implement the Action's NodeJS functionality.

Once these two steps are completed, you can use the Action from anywhere in your code.

### Declaring Actions[​](#declaring-actions "Direct link to Declaring Actions")

To create an Action in Wasp, we begin with an `action` declaration. Let's declare two Actions - one for creating a task, and another for marking tasks as done:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...action createTask {  fn: import { createTask } from "@src/actions.js"}action markTaskAsDone {  fn: import { markTaskAsDone } from "@src/actions.js"}
```

main.wasp

```
    // ...action createTask {  fn: import { createTask } from "@src/actions.js"}action markTaskAsDone {  fn: import { markTaskAsDone } from "@src/actions.js"}
```

If you want to know about all supported options for the `action` declaration, take a look at the [API Reference](#api-reference).

The names of Wasp Actions and their implementations don't necessarily have to match. However, to avoid confusion, we'll keep them the same.

info

You might have noticed that we told Wasp to import Action implementations that don't yet exist. Don't worry about that for now. We'll write the implementations imported from `actions.ts` in the next section.

It's a good idea to start with the high-level concept (the Action declaration in the Wasp file) and only then deal with the implementation details (the Action's implementation in JavaScript).

After declaring a Wasp Action, two important things happen:

-   Wasp **generates a server-side NodeJS function** that shares its name with the Action.
    
-   Wasp **generates a client-side JavaScript function** that shares its name with the Action (e.g., `markTaskAsDone`). This function takes a single optional argument - an object containing any serializable data you wish to use inside the Action. Wasp will send this object over the network and pass it into the Action's implementation as its first positional argument (more on this when we look at the implementations). Such an abstraction works thanks to an HTTP API route handler Wasp generates on the server, which calls the Action's NodeJS implementation under the hood.
    

Generating these two functions ensures a similar calling interface across the entire app (both client and server).

### Implementing Actions in Node[​](#implementing-actions-in-node "Direct link to Implementing Actions in Node")

Now that we've declared the Action, what remains is to implement it. We've instructed Wasp to look for the Actions' implementations in the file `src/actions.ts`, so that's where we should export them from.

Here's how you might implement the previously declared Actions `createTask` and `markTaskAsDone`:

-   JavaScript
-   TypeScript

src/actions.js

```
    // our "database"let nextId = 4const tasks = [  { id: 1, description: 'Buy some eggs', isDone: true },  { id: 2, description: 'Make an omelette', isDone: false },  { id: 3, description: 'Eat breakfast', isDone: false },]// You don't need to use the arguments if you don't need themexport const createTask = (args) => {  const newTask = {    id: nextId,    isDone: false,    description: args.description,  }  nextId += 1  tasks.push(newTask)  return newTask}// The 'args' object is something sent by the caller (most often from the client)export const markTaskAsDone = (args) => {  const task = tasks.find((task) => task.id === args.id)  if (!task) {    // We'll show how to properly handle such errors later    return  }  task.isDone = true}
```

Payload constraints

Wasp uses [superjson](https://github.com/blitz-js/superjson) under the hood. This means you're not limited to only sending and receiving JSON payloads.

You can send and receive any superjson-compatible payload (like Dates, Sets, Lists, circular references, etc.) and let Wasp handle the (de)serialization.

src/actions.ts

```
    import { type CreateTask, type MarkTaskAsDone } from 'wasp/server/operations'type Task = {  id: number  description: string  isDone: boolean}// our "database"let nextId = 4const tasks = [  { id: 1, description: 'Buy some eggs', isDone: true },  { id: 2, description: 'Make an omelette', isDone: false },  { id: 3, description: 'Eat breakfast', isDone: false },]// You don't need to use the arguments if you don't need themexport const createTask: CreateTask<Pick<Task, 'description'>, Task> = (  args) => {  const newTask = {    id: nextId,    isDone: false,    description: args.description,  }  nextId += 1  tasks.push(newTask)  return newTask}// The 'args' object is something sent by the caller (most often from the client)export const markTaskAsDone: MarkTaskAsDone<Pick<Task, 'id'>, void> = (  args) => {  const task = tasks.find((task) => task.id === args.id)  if (!task) {    // We'll show how to properly handle such errors later    return  }  task.isDone = true}
```

Payload constraints

Wasp uses [superjson](https://github.com/blitz-js/superjson) under the hood. This means you're not limited to only sending and receiving JSON payloads.

You can send and receive any superjson-compatible payload (like Dates, Sets, Lists, circular references, etc.) and let Wasp handle the (de)serialization.

#### Type support for Actions[​](#type-support-for-actions "Direct link to Type support for Actions")

Wasp automatically generates the types `CreateTask` and `MarkTaskAsDone` based on the declarations in your Wasp file:

-   `CreateTask` is a generic type that Wasp automatically generated based on the Action declaration for `createTask`.
-   `MarkTaskAsDone` is a generic type that Wasp automatically generated based on the Action declaration for `markTaskAsDone`.

Use these types to type the Action's implementation. It's optional but very helpful since doing so properly types the Action's context.

In this case, TypeScript will know the `context.entities` object must include the `Task` entity. TypeScript also knows whether the `context` object includes user information (it depends on whether your Action uses auth).

The generated types are generic and accept two optional type arguments: `Input` and `Output`.

1.  `Input` - The argument (the payload) received by the Action function.
2.  `Output` - The Action function's return type.

Use these type arguments to type the Action's inputs and outputs.

Explanation for the example above

The above code says that the Action `createTask` expects an object with the new task's description (its input type is `Pick<Task, 'description'>`) and returns the new task (its output type is `Task`).

On the other hand, the Action `markTaskAsDone` expects an object of type `Pick<Task, 'id'>`. This type is derived from the `Task` entity type.

If you don't care about typing the Action's inputs and outputs, you can omit both type arguments. TypeScript will then infer the most general types (`never` for the input and `unknown` for the output).

Specifying `Input` or `Output` is completely optional, but we highly recommended it. Doing so gives you:

-   Type support for the arguments and the return value inside the implementation.
-   **Full-stack type safety**. We'll explore what this means when we discuss calling the Action from the client.

Read more about type support for implementing Actions in the [API Reference](#implementing-actions).

Inferring the return type

If don't want to explicitly type the Action's return value, the `satisfies` keyword tells TypeScript to infer it automatically:

```
    const createFoo = (async (_args, context) => {  const foo = await context.entities.Foo.create()  return {    newFoo: foo,    message: "Here's your foo!",    returnedAt: new Date(),  }}) satisfies GetFoo
```

From the snippet above, TypeScript knows:

1.  The correct type for `context`.
2.  The Action's return type is `{ newFoo: Foo, message: string, returnedAt: Date }`.

If you don't need the context, you can skip specifying the Action's type (and arguments):

```
    const createFoo = () => {{ name: 'Foo', date: new Date() }}
```

For a detailed explanation of the Action definition API (more precisely, its arguments and return values), check the [API Reference](#api-reference).

### Using Actions[​](#using-actions "Direct link to Using Actions")

#### Using Actions on the client[​](#using-actions-on-the-client "Direct link to Using Actions on the client")

To call an Action on the client, you can import it from `wasp/client/operations` and call it directly.

The usage doesn't depend on whether the Action is authenticated or not. Wasp authenticates the logged-in user in the background.

-   JavaScript
-   TypeScript

```
    import { createTask, markTasAsDone } from 'wasp/client/operations'// ...const newTask = await createTask({ description: 'Learn TypeScript' })await markTasAsDone({ id: 1 })
```

```
    import { createTask, markTasAsDone } from 'wasp/client/operations'// TypeScript automatically infers the return values and type-checks// the payloads.const newTask = await createTask({ description: 'Keep learning TypeScript' })await markTasAsDone({ id: 1 })
```

Wasp supports **automatic full-stack type safety**. You only need to specify the Action's type in its server-side definition, and the client code will automatically know its API payload types.

When using Actions on the client, you'll most likely want to use them inside a component:

-   JavaScript
-   TypeScript

src/pages/Task.jsx

```
    import React from 'react'import { useQuery, getTask, markTaskAsDone } from 'wasp/client/operations'export const TaskPage = ({ id }) => {  const { data: task } = useQuery(getTask, { id })  if (!task) {    return <h1>"Loading"</h1>  }  const { description, isDone } = task  return (    <div>      <p>        <strong>Description: </strong>        {description}      </p>      <p>        <strong>Is done: </strong>        {isDone ? 'Yes' : 'No'}      </p>      {isDone || (        <button onClick={() => markTaskAsDone({ id })}>Mark as done.</button>      )}    </div>  )}
```

src/pages/Task.tsx

```
    import React from 'react'import { useQuery, getTask, markTaskAsDone } from 'wasp/client/operations'export const TaskPage = ({ id }: { id: number }) => {  const { data: task } = useQuery(getTask, { id })  if (!task) {    return <h1>"Loading"</h1>  }  const { description, isDone } = task  return (    <div>      <p>        <strong>Description: </strong>        {description}      </p>      <p>        <strong>Is done: </strong>        {isDone ? 'Yes' : 'No'}      </p>      {isDone || (        <button onClick={() => markTaskAsDone({ id })}>Mark as done.</button>      )}    </div>  )}
```

Since Actions don't require reactivity, they are safe to use inside components without a hook. Still, Wasp provides comes with the `useAction` hook you can use to enhance actions. Read all about it in the [API Reference](#api-reference).

#### Using Actions on the server[​](#using-actions-on-the-server "Direct link to Using Actions on the server")

Calling an Action on the server is similar to calling it on the client.

Here's what you have to do differently:

-   Import Actions from `wasp/server/operations` instead of `wasp/client/operations`.
-   Make sure you pass in a context object with the user to authenticated Actions.

-   JavaScript
-   TypeScript

```
    import { createTask, markTasAsDone } from 'wasp/server/operations'const user = // Get an AuthUser object, e.g., from context.userconst newTask = await createTask(  { description: 'Learn TypeScript' },  { user },)await markTasAsDone({ id: 1 }, { user })
```

```
    import { createTask, markTasAsDone } from 'wasp/server/operations'const user = // Get an AuthUser object, e.g., from context.user// TypeScript automatically infers the return values and type-checks// the payloads.const newTask = await createTask(  { description: 'Keep learning TypeScript' },  { user },)await markTasAsDone({ id: 1 }, { user })
```

### Error Handling[​](#error-handling "Direct link to Error Handling")

For security reasons, all exceptions thrown in the Action's NodeJS implementation are sent to the client as responses with the HTTP status code `500`, with all other details removed. Hiding error details by default helps against accidentally leaking possibly sensitive information over the network.

If you do want to pass additional error information to the client, you can construct and throw an appropriate `HttpError` in your implementation:

-   JavaScript
-   TypeScript

src/actions.js

```
    import { HttpError } from 'wasp/server'export const createTask = async (args, context) => {  throw new HttpError(    403, // status code    "You can't do this!", // message    { foo: 'bar' } // data  )}
```

src/actions.ts

```
    import { type CreateTask } from 'wasp/server/operations'import { HttpError } from 'wasp/server'export const createTask: CreateTask = async (args, context) => {  throw new HttpError(    403, // status code    "You can't do this!", // message    { foo: 'bar' } // data  )}
```

### Using Entities in Actions[​](#using-entities-in-actions "Direct link to Using Entities in Actions")

In most cases, resources used in Actions will be [Entities](/docs/data-model/entities). To use an Entity in your Action, add it to the `action` declaration in Wasp:

-   JavaScript
-   TypeScript

main.wasp

```
    action createTask {  fn: import { createTask } from "@src/actions.js",  entities: [Task]}action markTaskAsDone {  fn: import { markTaskAsDone } from "@src/actions.js",  entities: [Task]}
```

main.wasp

```
    action createTask {  fn: import { createTask } from "@src/actions.js",  entities: [Task]}action markTaskAsDone {  fn: import { markTaskAsDone } from "@src/actions.js",  entities: [Task]}
```

Wasp will inject the specified Entity into the Action's `context` argument, giving you access to the Entity's Prisma API. Wasp invalidates frontend Query caches by looking at the Entities used by each Action/Query. Read more about Wasp's smart cache invalidation [here](#cache-invalidation).

-   JavaScript
-   TypeScript

src/actions.js

```
    // The 'args' object is the payload sent by the caller (most often from the client)export const createTask = async (args, context) => {  const newTask = await context.entities.Task.create({    data: {      description: args.description,      isDone: false,    },  })  return newTask}export const markTaskAsDone = async (args, context) => {  await context.entities.Task.update({    where: { id: args.id },    data: { isDone: true },  })}
```

src/actions.ts

```
    import { type CreateTask, type MarkTaskAsDone } from 'wasp/server/operations'import { type Task } from 'wasp/entities'// The 'args' object is the payload sent by the caller (most often from the client)export const createTask: CreateTask<Pick<Task, 'description'>, Task> = async (  args,  context) => {  const newTask = await context.entities.Task.create({    data: {      description: args.description,      isDone: false,    },  })  return newTask}export const markTaskAsDone: MarkTaskAsDone<Pick<Task, 'id'>, void> = async (  args,  context) => {  await context.entities.Task.update({    where: { id: args.id },    data: { isDone: true },  })}
```

Again, annotating the Actions is optional, but greatly improves **full-stack type safety**.

The object `context.entities.Task` exposes `prisma.task` from [Prisma's CRUD API](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/crud).

## Cache Invalidation[​](#cache-invalidation "Direct link to Cache Invalidation")

One of the trickiest parts of managing a web app's state is making sure the data returned by the Queries is up to date. Since Wasp uses *react-query* for Query management, we must make sure to invalidate Queries (more specifically, their cached results managed by *react-query*) whenever they become stale.

It's possible to invalidate the caches manually through several mechanisms *react-query* provides (e.g., refetch, direct invalidation). However, since manual cache invalidation quickly becomes complex and error-prone, Wasp offers a faster and a more effective solution to get you started: **automatic Entity-based Query cache invalidation**. Because Actions can (and most often do) modify the state while Queries read it, Wasp invalidates a Query's cache whenever an Action that uses the same Entity is executed.

For example, if the Action `createTask` and Query `getTasks` both use the Entity `Task`, executing `createTask` may cause the cached result of `getTasks` to become outdated. In response, Wasp will invalidate it, causing `getTasks` to refetch data from the server and update it.

In practice, this means that Wasp keeps the Queries "fresh" without requiring you to think about cache invalidation.

On the other hand, this kind of automatic cache invalidation can become wasteful (some updates might not be necessary) and will only work for Entities. If that's an issue, you can use the mechanisms provided by *react-query* for now, and expect more direct support in Wasp for handling those use cases in a nice, elegant way.

If you wish to optimistically set cache values after performing an Action, you can do so using [optimistic updates](https://stackoverflow.com/a/33009713). Configure them using Wasp's [useAction hook](#the-useaction-hook-and-optimistic-updates). This is currently the only manual cache invalidation mechanism Wasps supports natively. For everything else, you can always rely on *react-query*.

## Differences Between Queries and Actions[​](#differences-between-queries-and-actions "Direct link to Differences Between Queries and Actions")

Actions and Queries are two closely related concepts in Wasp. They might seem to perform similar tasks, but Wasp treats them differently, and each concept represents a different thing.

Here are the key differences between Queries and Actions:

1.  Actions can (and often should) modify the server's state, while Queries are only permitted to read it. Wasp relies on you adhering to this convention when performing cache invalidations, so it's crucial to follow it.
2.  Actions don't need to be reactive, so you can call them directly. However, Wasp does provide a [`useAction` React hook](#the-useaction-hook-and-optimistic-updates) for adding extra behavior to the Action (like optimistic updates).
3.  `action` declarations in Wasp are mostly identical to `query` declarations. The only difference lies in the declaration's name.

## API Reference[​](#api-reference "Direct link to API Reference")

### Declaring Actions in Wasp[​](#declaring-actions-in-wasp "Direct link to Declaring Actions in Wasp")

The `action` declaration supports the following fields:

-   `fn: ExtImport` required
    
    The import statement of the Action's NodeJs implementation.
    
-   `entities: [Entity]`
    
    A list of entities you wish to use inside your Action. For instructions on using Entities in Actions, take a look at [the guide](#using-entities-in-actions).
    

#### Example[​](#example "Direct link to Example")

-   JavaScript
-   TypeScript

Declaring the Action:

```
    query createFoo {    fn: import { createFoo } from "@src/actions.js"    entities: [Foo]}
```

Enables you to import and use it anywhere in your code (on the server or the client):

```
    // Use it on the clientimport { createFoo } from 'wasp/client/operations'// Use it on the serverimport { createFoo } from 'wasp/server/operations'
```

Declaring the Action:

```
    query createFoo {    fn: import { createFoo } from "@src/actions.js"    entities: [Foo]}
```

Enables you to import and use it anywhere in your code (on the server or the client):

```
    // Use it on the clientimport { createFoo } from 'wasp/client/operations'// Use it on the serverimport { createFoo } from 'wasp/server/operations'
```

As well as the following type import on the server:

```
    import { type CreateFoo } from 'wasp/server/operations'
```

### Implementing Actions[​](#implementing-actions "Direct link to Implementing Actions")

The Action's implementation is a NodeJS function that takes two arguments (it can be an `async` function if you need to use the `await` keyword). Since both arguments are positional, you can name the parameters however you want, but we'll stick with `args` and `context`:

1.  `args` (type depends on the Action)
    
    An object containing the data **passed in when calling the Action** (e.g., filtering conditions). Check [the usage examples](#using-actions) to see how to pass this object to the Action.
    
2.  `context` (type depends on the Action)
    
    An additional context object **passed into the Action by Wasp**. This object contains user session information, as well as information about entities. Check the [section about using entities in Actions](#using-entities-in-actions) to see how to use the entities field on the `context` object, or the [auth section](/docs/auth/overview#using-the-contextuser-object) to see how to use the `user` object.
    

#### Example[​](#example-1 "Direct link to Example")

-   JavaScript
-   TypeScript

The following Action:

```
    action createFoo {    fn: import { createFoo } from "@src/actions.js"    entities: [Foo]}
```

Expects to find a named export `createfoo` from the file `src/actions.js`

actions.js

```
    export const createFoo = (args, context) => {  // implementation}
```

The following Action:

```
    action createFoo {    fn: import { createFoo } from "@src/actions.js"    entities: [Foo]}
```

Expects to find a named export `createfoo` from the file `src/actions.js`

You can use the generated type `CreateFoo` and specify the Action's inputs and outputs using its type arguments.

actions.ts

```
    import { type CreateFoo } from 'wasp/server/operations'type Foo = // ...export const createFoo: CreateFoo<{ bar: string }, Foo> = (args, context) => {  // implementation};
```

In this case, the Action expects to receive an object with a `bar` field of type `string` (this is the type of `args`), and return a value of type `Foo` (this must match the type of the Action's return value).

### The `useAction` Hook and Optimistic Updates[​](#the-useaction-hook-and-optimistic-updates "Direct link to the-useaction-hook-and-optimistic-updates")

Make sure you understand how [Queries](/docs/data-model/operations/queries) and [Cache Invalidation](#cache-invalidation) work before reading this chapter.

When using Actions in components, you can enhance them with the help of the `useAction` hook. This hook comes bundled with Wasp, and is used for decorating Wasp Actions. In other words, the hook returns a function whose API matches the original Action while also doing something extra under the hood (depending on how you configure it).

The `useAction` hook accepts two arguments:

-   `actionFn` required
    
    The Wasp Action (the client-side Action function generated by Wasp based on a Action declaration) you wish to enhance.
    
-   `actionOptions`
    
    An object configuring the extra features you want to add to the given Action. While this argument is technically optional, there is no point in using the `useAction` hook without providing it (it would be the same as using the Action directly). The Action options object supports the following fields:
    
    -   `optimisticUpdates`
        
        An array of objects where each object defines an [optimistic update](https://stackoverflow.com/a/33009713) to perform on the Query cache. To define an optimistic update, you must specify the following properties:
        
        -   `getQuerySpecifier` required
        
        A function returning the Query specifier (a value used to address the Query you want to update). A Query specifier is an array specifying the query function and arguments. For example, to optimistically update the Query used with `useQuery(fetchFilteredTasks, {isDone: true }]`, your `getQuerySpecifier` function would have to return the array `[fetchFilteredTasks, { isDone: true}]`. Wasp will forward the argument you pass into the decorated Action to this function (you can use the properties of the added/changed item to address the Query).
        
        -   `updateQuery` required
        
        The function used to perform the optimistic update. It should return the desired state of the cache. Wasp will call it with the following arguments:
        
        -   `item` - The argument you pass into the decorated Action.
        -   `oldData` - The currently cached value for the Query identified by the specifier.

caution

The `updateQuery` function must be a pure function. It must return the desired cache value identified by the `getQuerySpecifier` function and *must not* perform any side effects.

Also, make sure you only update the Query caches affected by your Action causing the optimistic update (Wasp cannot yet verify this).

Finally, your implementation of the `updateQuery` function should work correctly regardless of the state of `oldData` (e.g., don't rely on array positioning). If you need to do something else during your optimistic update, you can directly use *react-query*'s lower-level API (read more about it [here](#advanced-usage)).

Here's an example showing how to configure the Action `markTaskAsDone` that toggles a task's `isDone` status to perform an optimistic update:

-   JavaScript
-   TypeScript

src/pages/Task.jsx

```
    import React from 'react'import {  useQuery,  useAction,  getTask,  markTaskAsDone,} from 'wasp/client/operations'const TaskPage = ({ id }) => {  const { data: task } = useQuery(getTask, { id })  const markTaskAsDoneOptimistically = useAction(markTaskAsDone, {    optimisticUpdates: [      {        getQuerySpecifier: ({ id }) => [getTask, { id }],        updateQuery: (_payload, oldData) => ({ ...oldData, isDone: true }),      },    ],  })  if (!task) {    return <h1>"Loading"</h1>  }  const { description, isDone } = task  return (    <div>      <p>        <strong>Description: </strong>        {description}      </p>      <p>        <strong>Is done: </strong>        {isDone ? 'Yes' : 'No'}      </p>      {isDone || (        <button onClick={() => markTaskAsDoneOptimistically({ id })}>          Mark as done.        </button>      )}    </div>  )}export default TaskPage
```

src/pages/Task.tsx

```
    import React from 'react'import {  useQuery,  useAction,  type OptimisticUpdateDefinition,  getTask,  markTaskAsDone,} from 'wasp/client/operations'type TaskPayload = Pick<Task, "id">;const TaskPage = ({ id }: { id: number }) => {  const { data: task } = useQuery(getTask, { id });  // Typescript automatically type-checks the payload type.  const markTaskAsDoneOptimistically = useAction(markTaskAsDone, {    optimisticUpdates: [      {        getQuerySpecifier: ({ id }) => [getTask, { id }],        updateQuery: (_payload, oldData) => ({ ...oldData, isDone: true }),      } as OptimisticUpdateDefinition<TaskPayload, Task>,    ],  });  if (!task) {    return <h1>"Loading"</h1>;  }  const { description, isDone } = task;  return (    <div>      <p>        <strong>Description: </strong>        {description}      </p>      <p>        <strong>Is done: </strong>        {isDone ? "Yes" : "No"}      </p>      {isDone || (        <button onClick={() => markTaskAsDoneOptimistically({ id })}>          Mark as done.        </button>      )}    </div>  );};export default TaskPage;
```

#### Advanced usage[​](#advanced-usage "Direct link to Advanced usage")

The `useAction` hook currently only supports specifying optimistic updates. You can expect more features in future versions of Wasp.

Wasp's optimistic update API is deliberately small and focuses exclusively on updating Query caches (as that's the most common use case). You might need an API that offers more options or a higher level of control. If that's the case, instead of using Wasp's `useAction` hook, you can use *react-query*'s `useMutation` hook and directly work with [their low-level API](https://tanstack.com/query/v4/docs/framework/react/guides/optimistic-updates).

If you decide to use *react-query*'s API directly, you will need access to Query cache key. Wasp internally uses this key but abstracts it from the programmer. Still, you can easily obtain it by accessing the `queryCacheKey` property on any Query:

-   JavaScript
-   TypeScript

```
    import { getTasks } from 'wasp/client/operations'const queryKey = getTasks.queryCacheKey
```

```
    import { getTasks } from 'wasp/client/operations'const queryKey = getTasks.queryCacheKey
```

---

# Automatic CRUD

If you have a lot of experience writing full-stack apps, you probably ended up doing some of the same things many times: listing data, adding data, editing it, and deleting it.

Wasp makes handling these boring bits easy by offering a higher-level concept called Automatic CRUD.

With a single declaration, you can tell Wasp to automatically generate server-side logic (i.e., Queries and Actions) for creating, reading, updating and deleting [Entities](/docs/data-model/entities). As you update definitions for your Entities, Wasp automatically regenerates the backend logic.

Early preview

This feature is currently in early preview and we are actively working on it. Read more about [our plans](#future-of-crud-operations-in-wasp) for CRUD operations.

## Overview[​](#overview "Direct link to Overview")

Imagine we have a `Task` entity and we want to enable CRUD operations for it:

schema.prisma

```
    model Task {  id          Int     @id @default(autoincrement())  description String  isDone      Boolean}
```

We can then define a new `crud` called `Tasks`.

We specify to use the `Task` entity and we enable the `getAll`, `get`, `create` and `update` operations (let's say we don't need the `delete` operation).

main.wasp

```
    crud Tasks {  entity: Task,  operations: {    getAll: {      isPublic: true, // by default only logged in users can perform operations    },    get: {},    create: {      overrideFn: import { createTask } from "@src/tasks.js",    },    update: {},  },}
```

1.  It uses default implementation for `getAll`, `get`, and `update`,
2.  ... while specifying a custom implementation for `create`.
3.  `getAll` will be public (no auth needed), while the rest of the operations will be private.

Here's what it looks like when visualized:

![Automatic CRUD with Wasp](/img/crud_diagram.png)

Visualization of the Tasks crud declaration

We can now use the CRUD queries and actions we just specified in our client code.

Keep reading for an example of Automatic CRUD in action, or skip ahead for the [API Reference](#api-reference).

## Example: A Simple TODO App[​](#example-a-simple-todo-app "Direct link to Example: A Simple TODO App")

Let's create a full-app example that uses automatic CRUD. We'll stick to using the `Task` entity from the previous example, but we'll add a `User` entity and enable [username and password](/docs/auth/username-and-pass) based auth.

![Automatic CRUD with Wasp](/img/crud-guide.gif)

We are building a simple tasks app with username based auth

### Creating the App[​](#creating-the-app "Direct link to Creating the App")

We can start by running `wasp new tasksCrudApp` and then adding the following to the `main.wasp` file:

main.wasp

```
    app tasksCrudApp {  wasp: {    version: "^0.13.0"  },  title: "Tasks Crud App",  // We enabled auth and set the auth method to username and password  auth: {    userEntity: User,    methods: {      usernameAndPassword: {},    },    onAuthFailedRedirectTo: "/login",  },}// Tasks app routesroute RootRoute { path: "/", to: MainPage }page MainPage {  component: import { MainPage } from "@src/MainPage.jsx",  authRequired: true,}route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { LoginPage } from "@src/LoginPage.jsx",}route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { SignupPage } from "@src/SignupPage.jsx",}
```

And let's define our entities in the `schema.prisma` file:

schema.prisma

```
    model User {  id    Int    @id @default(autoincrement())  tasks Task[]}// We defined a Task entity on which we'll enable CRUD later onmodel Task {  id          Int     @id @default(autoincrement())  description String  isDone      Boolean  userId      Int  user        User    @relation(fields: [userId], references: [id])}
```

We can then run `wasp db migrate-dev` to create the database and run the migrations.

### Adding CRUD to the `Task` Entity ✨[​](#adding-crud-to-the-task-entity- "Direct link to adding-crud-to-the-task-entity-")

Let's add the following `crud` declaration to our `main.wasp` file:

main.wasp

```
    // ...crud Tasks {  entity: Task,  operations: {    getAll: {},    create: {      overrideFn: import { createTask } from "@src/tasks.js",    },  },}
```

You'll notice that we enabled only `getAll` and `create` operations. This means that only these operations will be available.

We also overrode the `create` operation with a custom implementation. This means that the `create` operation will not be generated, but instead, the `createTask` function from `@src/tasks.ts` will be used.

### Our Custom `create` Operation[​](#our-custom-create-operation "Direct link to our-custom-create-operation")

We need a custom `create` operation because we want to make sure that the task is connected to the user creating it. Automatic CRUD doesn't yet support this by default. Read more about the default implementations [here](#declaring-a-crud-with-default-options).

Here's the `src/tasks.ts` file:

-   JavaScript
-   TypeScript

src/tasks.js

```
    import { HttpError } from 'wasp/server'export const createTask = async (args, context) => {  if (!context.user) {    throw new HttpError(401, 'User not authenticated.')  }  const { description, isDone } = args  const { Task } = context.entities  return await Task.create({    data: {      description,      isDone,      // Connect the task to the user that is creating it      user: {        connect: {          id: context.user.id,        },      },    },  })}
```

src/tasks.ts

```
    import { type Tasks } from 'wasp/server/crud'import { type Task } from 'wasp/entities'import { HttpError } from 'wasp/server'type CreateTaskInput = { description: string; isDone: boolean }export const createTask: Tasks.CreateAction<CreateTaskInput, Task> = async (  args,  context) => {  if (!context.user) {    throw new HttpError(401, 'User not authenticated.')  }  const { description, isDone } = args  const { Task } = context.entities  return await Task.create({    data: {      description,      isDone,      // Connect the task to the user that is creating it      user: {        connect: {          id: context.user.id,        },      },    },  })}
```

Wasp automatically generates the `Tasks.CreateAction` type based on the CRUD declaration in your Wasp file. Use it to type the CRUD action's implementation.

The `Tasks.CreateAction` type works exactly like the types Wasp generates for [Queries](/docs/data-model/operations/queries#type-support-for-queries) and [Actions](/docs/data-model/operations/actions#type-support-for-actions). In other words, annotating the action with `Tasks.CreateAction` tells TypeScript about the type of the Action's `context` object, while the two type arguments allow you to specify the Action's inputs and outputs.

Read more about type support for CRUD overrides in the [API reference](#defining-the-overrides).

### Using the Generated CRUD Operations on the Client[​](#using-the-generated-crud-operations-on-the-client "Direct link to Using the Generated CRUD Operations on the Client")

And let's use the generated operations in our client code:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import { Tasks } from 'wasp/client/crud'import { useState } from 'react'export const MainPage = () => {  const { data: tasks, isLoading, error } = Tasks.getAll.useQuery()  const createTask = Tasks.create.useAction()  const [taskDescription, setTaskDescription] = useState('')  function handleCreateTask() {    createTask({ description: taskDescription, isDone: false })    setTaskDescription('')  }  if (isLoading) return <div>Loading...</div>  if (error) return <div>Error: {error.message}</div>  return (    <div      style={{        fontSize: '1.5rem',        display: 'grid',        placeContent: 'center',        height: '100vh',      }}    >      <div>        <input          value={taskDescription}          onChange={(e) => setTaskDescription(e.target.value)}        />        <button onClick={handleCreateTask}>Create task</button>      </div>      <ul>        {tasks.map((task) => (          <li key={task.id}>{task.description}</li>        ))}      </ul>    </div>  )}
```

src/MainPage.tsx

```
    import { Tasks } from 'wasp/client/crud'import { useState } from 'react'export const MainPage = () => {  // Thanks to full-stack type safety, all payload types are inferred  // automatically  const { data: tasks, isLoading, error } = Tasks.getAll.useQuery()  const createTask = Tasks.create.useAction()  const [taskDescription, setTaskDescription] = useState('')  function handleCreateTask() {    createTask({ description: taskDescription, isDone: false })    setTaskDescription('')  }  if (isLoading) return <div>Loading...</div>  if (error) return <div>Error: {error.message}</div>  return (    <div      style={{        fontSize: '1.5rem',        display: 'grid',        placeContent: 'center',        height: '100vh',      }}    >      <div>        <input          value={taskDescription}          onChange={(e) => setTaskDescription(e.target.value)}        />        <button onClick={handleCreateTask}>Create task</button>      </div>      <ul>        {tasks.map((task) => (          <li key={task.id}>{task.description}</li>        ))}      </ul>    </div>  )}
```

And here are the login and signup pages, where we are using Wasp's [Auth UI](/docs/auth/ui) components:

-   JavaScript
-   TypeScript

src/LoginPage.jsx

```
    import { LoginForm } from 'wasp/client/auth'import { Link } from 'react-router-dom'export function LoginPage() {  return (    <div      style={{        display: 'grid',        placeContent: 'center',      }}    >      <LoginForm />      <div>        <Link to="/signup">Create an account</Link>      </div>    </div>  )}
```

src/LoginPage.tsx

```
    import { LoginForm } from 'wasp/client/auth'import { Link } from 'react-router-dom'export function LoginPage() {  return (    <div      style={{        display: 'grid',        placeContent: 'center',      }}    >      <LoginForm />      <div>        <Link to="/signup">Create an account</Link>      </div>    </div>  )}
```

-   JavaScript
-   TypeScript

src/SignupPage.jsx

```
    import { SignupForm } from 'wasp/client/auth'export function SignupPage() {  return (    <div      style={{        display: 'grid',        placeContent: 'center',      }}    >      <SignupForm />    </div>  )}
```

src/SignupPage.tsx

```
    import { SignupForm } from 'wasp/client/auth'export function SignupPage() {  return (    <div      style={{        display: 'grid',        placeContent: 'center',      }}    >      <SignupForm />    </div>  )}
```

That's it. You can now run `wasp start` and see the app in action. ⚡️

You should see a login page and a signup page. After you log in, you should see a page with a list of tasks and a form to create new tasks.

## Future of CRUD Operations in Wasp[​](#future-of-crud-operations-in-wasp "Direct link to Future of CRUD Operations in Wasp")

CRUD operations currently have a limited set of knowledge about the business logic they are implementing.

-   For example, they don't know that a task should be connected to the user that is creating it. This is why we had to override the `create` operation in the example above.
-   Another thing: they are not aware of the authorization rules. For example, they don't know that a user should not be able to create a task for another user. In the future, we will be adding role-based authorization to Wasp, and we plan to make CRUD operations aware of the authorization rules.
-   Another issue is input validation and sanitization. For example, we might want to make sure that the task description is not empty.

CRUD operations are a mechanism for getting a backend up and running quickly, but it depends on the information it can get from the Wasp app. The more information that it can pick up from your app, the more powerful it will be out of the box.

We plan on supporting CRUD operations and growing them to become the easiest way to create your backend. Follow along on [this GitHub issue](https://github.com/wasp-lang/wasp/issues/1253) to see how we are doing.

## API Reference[​](#api-reference "Direct link to API Reference")

CRUD declaration works on top of an existing entity declaration. We'll fully explore the API using two examples:

1.  A basic CRUD declaration that relies on default options.
2.  A more involved CRUD declaration that uses extra options and overrides.

### Declaring a CRUD With Default Options[​](#declaring-a-crud-with-default-options "Direct link to Declaring a CRUD With Default Options")

If we create CRUD operations for an entity named `Task`, like this:

-   JavaScript
-   TypeScript

main.wasp

```
    crud Tasks { // crud name here is "Tasks"  entity: Task,  operations: {    get: {},    getAll: {},    create: {},    update: {},    delete: {},  },}
```

Wasp will give you the following default implementations:

**get** - returns one entity based on the `id` field

```
    // ...// Wasp uses the field marked with `@id` in Prisma schema as the id field.return Task.findUnique({ where: { id: args.id } })
```

**getAll** - returns all entities

```
    // ...// If the operation is not public, Wasp checks if an authenticated user// is making the request.return Task.findMany()
```

**create** - creates a new entity

```
    // ...return Task.create({ data: args.data })
```

**update** - updates an existing entity

```
    // ...// Wasp uses the field marked with `@id` in Prisma schema as the id field.return Task.update({ where: { id: args.id }, data: args.data })
```

**delete** - deletes an existing entity

```
    // ...// Wasp uses the field marked with `@id` in Prisma schema as the id field.return Task.delete({ where: { id: args.id } })
```

main.wasp

```
    crud Tasks { // crud name here is "Tasks"  entity: Task,  operations: {    get: {},    getAll: {},    create: {},    update: {},    delete: {},  },}
```

Wasp will give you the following default implementations:

**get** - returns one entity based on the `id` field

```
    // ...// Wasp uses the field marked with `@id` in Prisma schema as the id field.return Task.findUnique({ where: { id: args.id } })
```

**getAll** - returns all entities

```
    // ...// If the operation is not public, Wasp checks if an authenticated user// is making the request.return Task.findMany()
```

**create** - creates a new entity

```
    // ...return Task.create({ data: args.data })
```

**update** - updates an existing entity

```
    // ...// Wasp uses the field marked with `@id` in Prisma schema as the id field.return Task.update({ where: { id: args.id }, data: args.data })
```

**delete** - deletes an existing entity

```
    // ...// Wasp uses the field marked with `@id` in Prisma schema as the id field.return Task.delete({ where: { id: args.id } })
```

Current Limitations

In the default `create` and `update` implementations, we are saving all of the data that the client sends to the server. This is not always desirable, i.e. in the case when the client should not be able to modify all of the data in the entity.

[In the future](#future-of-crud-operations-in-wasp), we are planning to add validation of action input, where only the data that the user is allowed to change will be saved.

For now, the solution is to provide an override function. You can override the default implementation by using the `overrideFn` option and implementing the validation logic yourself.

### Declaring a CRUD With All Available Options[​](#declaring-a-crud-with-all-available-options "Direct link to Declaring a CRUD With All Available Options")

Here's an example of a more complex CRUD declaration:

-   JavaScript
-   TypeScript

main.wasp

```
    crud Tasks { // crud name here is "Tasks"  entity: Task,  operations: {    getAll: {      isPublic: true, // optional, defaults to false    },    get: {},    create: {      overrideFn: import { createTask } from "@src/tasks.js", // optional    },    update: {},  },}
```

main.wasp

```
    crud Tasks { // crud name here is "Tasks"  entity: Task,  operations: {    getAll: {      isPublic: true, // optional, defaults to false    },    get: {},    create: {      overrideFn: import { createTask } from "@src/tasks.js", // optional    },    update: {},  },}
```

The CRUD declaration features the following fields:

-   `entity: Entity` required
    
    The entity to which the CRUD operations will be applied.
    
-   `operations: { [operationName]: CrudOperationOptions }` required
    
    The operations to be generated. The key is the name of the operation, and the value is the operation configuration.
    
    -   The possible values for `operationName` are:
        -   `getAll`
        -   `get`
        -   `create`
        -   `update`
        -   `delete`
    -   `CrudOperationOptions` can have the following fields:
        -   `isPublic: bool` - Whether the operation is public or not. If it is public, no auth is required to access it. If it is not public, it will be available only to authenticated users. Defaults to `false`.
        -   `overrideFn: ExtImport` - The import statement of the optional override implementation in Node.js.

#### Defining the overrides[​](#defining-the-overrides "Direct link to Defining the overrides")

Like with actions and queries, you can define the implementation in a Javascript/Typescript file. The overrides are functions that take the following arguments:

-   `args`
    
    The arguments of the operation i.e. the data sent from the client.
    
-   `context`
    
    Context contains the `user` making the request and the `entities` object with the entity that's being operated on.
    

For a usage example, check the [example guide](/docs/data-model/crud#adding-crud-to-the-task-entity-).

#### Using the CRUD operations in client code[​](#using-the-crud-operations-in-client-code "Direct link to Using the CRUD operations in client code")

On the client, you import the CRUD operations from `wasp/client/crud` by import the `{crud name}` object. For example, if you have a CRUD called `Tasks`, you would import the operations like this:

-   JavaScript
-   TypeScript

SomePage.jsx

```
    import { Tasks } from 'wasp/client/crud'
```

SomePage.tsx

```
    import { Tasks } from 'wasp/client/crud'
```

You can then access the operations like this:

-   JavaScript
-   TypeScript

SomePage.jsx

```
    const { data } = Tasks.getAll.useQuery()const { data } = Tasks.get.useQuery({ id: 1 })const createAction = Tasks.create.useAction()const updateAction = Tasks.update.useAction()const deleteAction = Tasks.delete.useAction()
```

SomePage.tsx

```
    const { data } = Tasks.getAll.useQuery()const { data } = Tasks.get.useQuery({ id: 1 })const createAction = Tasks.create.useAction()const updateAction = Tasks.update.useAction()const deleteAction = Tasks.delete.useAction()
```

All CRUD operations are implemented with [Queries and Actions](/docs/data-model/operations/overview) under the hood, which means they come with all the features you'd expect (e.g., automatic SuperJSON serialization, full-stack type safety when using TypeScript)

---

Join our **community** on [Discord](https://discord.com/invite/rzdnErX), where we chat about full-stack web stuff. Join us to see what we are up to, share your opinions or get help with CRUD operations.

---

# Databases

[Entities](/docs/data-model/entities), [Operations](/docs/data-model/operations/overview) and [Automatic CRUD](/docs/data-model/crud) together make a high-level interface for working with your app's data. Still, all that data has to live somewhere, so let's see how Wasp deals with databases.

## Supported Database Backends[​](#supported-database-backends "Direct link to Supported Database Backends")

Wasp supports multiple database backends. We'll list and explain each one.

### SQLite[​](#sqlite "Direct link to SQLite")

The default database Wasp uses is [SQLite](https://www.sqlite.org/index.html).

When you create a new Wasp project, the `schema.prisma` file will have SQLite as the default database provider:

schema.prisma

```
    datasource db {  provider = "sqlite"  url      = env("DATABASE_URL")}// ...
```

Read more about how Wasp uses the Prisma schema file in the [Prisma schema file](/docs/data-model/prisma-file) section.

When you use the SQLite database, Wasp sets the `DATABASE_URL` environment variable for you.

SQLite is a great way to get started with a new project because it doesn't require any configuration, but Wasp can only use it in development. Once you want to deploy your Wasp app to production, you'll need to switch to PostgreSQL and stick with it.

Fortunately, migrating from SQLite to PostgreSQL is pretty simple, and we have [a guide](#migrating-from-sqlite-to-postgresql) to help you.

### PostgreSQL[​](#postgresql "Direct link to PostgreSQL")

[PostgreSQL](https://www.postgresql.org/) is the most advanced open-source database and one of the most popular databases overall. It's been in active development for 20+ years. Therefore, if you're looking for a battle-tested database, look no further.

To use PostgreSQL with Wasp, set the provider to `"postgresql"` in the `schema.prisma` file:

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}// ...
```

Read more about how Wasp uses the Prisma schema file in the [Prisma schema file](/docs/data-model/prisma-file) section.

You'll have to ensure a database instance is running during development to use PostgreSQL. Wasp needs access to your database for commands such as `wasp start` or `wasp db migrate-dev`.

We cover all supported ways of connecting to a database in [the next section](#connecting-to-a-database).

## Connecting to a Database[​](#connecting-to-a-database "Direct link to Connecting to a Database")

### SQLite[​](#sqlite-1 "Direct link to SQLite")

If you are using SQLite, you don't need to do anything special to connect to the database. Wasp will take care of it for you.

### PostgreSQL[​](#postgresql-1 "Direct link to PostgreSQL")

If you are using PostgreSQL, Wasp supports two ways of connecting to a database:

1.  For managed experience, let Wasp spin up a ready-to-go development database for you.
2.  For more control, you can specify a database URL and connect to an existing database that you provisioned yourself.

#### Using the Dev Database provided by Wasp[​](#using-the-dev-database-provided-by-wasp "Direct link to Using the Dev Database provided by Wasp")

The command `wasp start db` will start a default PostgreSQL dev database for you.

Your Wasp app will automatically connect to it, just keep `wasp start db` running in the background. Also, make sure that:

-   You have [Docker installed](https://www.docker.com/get-started/) and it's available in your `PATH`.
-   The port `5432` isn't taken.

tip

In case you might want to connect to the dev database through the external tool like `psql` or [pgAdmin](https://www.pgadmin.org/), the credentials are printed in the console when you run `wasp db start`, at the very beginning.

#### Connecting to an existing database[​](#connecting-to-an-existing-database "Direct link to Connecting to an existing database")

If you want to spin up your own dev database (or connect to an external one), you can tell Wasp about it using the `DATABASE_URL` environment variable. Wasp will use the value of `DATABASE_URL` as a connection string.

The easiest way to set the necessary `DATABASE_URL` environment variable is by adding it to the [.env.server](/docs/project/env-vars) file in the root dir of your Wasp project (if that file doesn't yet exist, create it):

.env.server

```
    DATABASE_URL=postgresql://user:password@localhost:5432/mydb
```

Alternatively, you can set it inline when running `wasp` (this applies to all environment variables):

```
    DATABASE_URL=<my-db-url> wasp ...
```

This trick is useful for running a certain `wasp` command on a specific database. For example, you could do:

```
    DATABASE_URL=<production-db-url> wasp db seed myProductionSeed
```

This command seeds the data for a fresh staging or production database. Read more about [seeding the database](#seeding-the-database).

## Migrating from SQLite to PostgreSQL[​](#migrating-from-sqlite-to-postgresql "Direct link to Migrating from SQLite to PostgreSQL")

To run your Wasp app in production, you'll need to switch from SQLite to PostgreSQL.

1.  Set the provider to `"postgresql"` in the `schema.prisma` file:
    
    schema.prisma
    
    ```
        datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}// ...
    ```
    
2.  Delete all the old migrations, since they are SQLite migrations and can't be used with PostgreSQL, as well as the SQLite database by running [`wasp clean`](https://wasp-lang.dev/docs/general/cli#project-commands):
    
    ```
        rm -r migrations/wasp clean
    ```
    
3.  Ensure your new database is running (check the [section on connecting to a database](#connecting-to-a-database) to see how). Leave it running, since we need it for the next step.
    
4.  In a different terminal, run `wasp db migrate-dev` to apply the changes and create a new initial migration.
    
5.  That is it, you are all done!
    

## Seeding the Database[​](#seeding-the-database "Direct link to Seeding the Database")

**Database seeding** is a term used for populating the database with some initial data.

Seeding is most commonly used for:

1.  Getting the development database into a state convenient for working and testing.
2.  Initializing any database (`dev`, `staging`, or `prod`) with essential data it requires to operate. For example, populating the Currency table with default currencies, or the Country table with all available countries.

### Writing a Seed Function[​](#writing-a-seed-function "Direct link to Writing a Seed Function")

You can define as many **seed functions** as you want in an array under the `app.db.seeds` field:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  // ...  db: {    seeds: [      import { devSeedSimple } from "@src/dbSeeds.js",      import { prodSeed } from "@src/dbSeeds.js"    ]  }}
```

main.wasp

```
    app MyApp {  // ...  db: {    seeds: [      import { devSeedSimple } from "@src/dbSeeds.js",      import { prodSeed } from "@src/dbSeeds.js"    ]  }}
```

Each seed function must be an async function that takes one argument, `prisma`, which is a [Prisma Client](https://www.prisma.io/docs/concepts/components/prisma-client/crud) instance used to interact with the database. This is the same Prisma Client instance that Wasp uses internally.

Since a seed function falls under server-side code, it can import other server-side functions. This is convenient because you might want to seed the database using Actions.

Here's an example of a seed function that imports an Action:

-   JavaScript
-   TypeScript

```
    import { createTask } from './actions.js'import { sanitizeAndSerializeProviderData } from 'wasp/server/auth'export const devSeedSimple = async (prisma) => {  const user = await createUser(prisma, {    username: 'RiuTheDog',    password: 'bark1234',  })  await createTask(    { description: 'Chase the cat' },    { user, entities: { Task: prisma.task } }  )}async function createUser(prisma, data) {  const newUser = await prisma.user.create({    data: {      auth: {        create: {          identities: {            create: {              providerName: 'username',              providerUserId: data.username,              providerData: sanitizeAndSerializeProviderData({                hashedPassword: data.password              }),            },          },        },      },    },  })  return newUser}
```

```
    import { createTask } from './actions.js'import { type DbSeedFn } from 'wasp/server'import { sanitizeAndSerializeProviderData } from 'wasp/server/auth'import { type AuthUser } from 'wasp/auth'import { PrismaClient } from '@prisma/client'export const devSeedSimple: DbSeedFn = async (prisma) => {  const user = await createUser(prisma, {    username: 'RiuTheDog',    password: 'bark1234',  })  await createTask(    { description: 'Chase the cat', isDone: false },    { user, entities: { Task: prisma.task } }  )};async function createUser(  prisma: PrismaClient,  data: { username: string, password: string }): Promise<AuthUser> {  const newUser = await prisma.user.create({    data: {      auth: {        create: {          identities: {            create: {              providerName: 'username',              providerUserId: data.username,              providerData: sanitizeAndSerializeProviderData<'username'>({                hashedPassword: data.password              }),            },          },        },      },    },  })  return newUser}
```

Wasp exports a type called `DbSeedFn` which you can use to easily type your seeding function. Wasp defines `DbSeedFn` like this:

```
    type DbSeedFn = (prisma: PrismaClient) => Promise<void>
```

Annotating the function `devSeedSimple` with this type tells TypeScript:

-   The seeding function's argument (`prisma`) is of type `PrismaClient`.
-   The seeding function's return value is `Promise<void>`.

### Running seed functions[​](#running-seed-functions "Direct link to Running seed functions")

Run the command `wasp db seed` and Wasp will ask you which seed function you'd like to run (if you've defined more than one).

Alternatively, run the command `wasp db seed <seed-name>` to choose a specific seed function right away, for example:

```
    wasp db seed devSeedSimple
```

Check the [API Reference](#cli-commands-for-seeding-the-database) for more details on these commands.

tip

You'll often want to call `wasp db seed` right after you run `wasp db reset`, as it makes sense to fill the database with initial data after clearing it.

## API Reference[​](#api-reference "Direct link to API Reference")

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  // ...  db: {    seeds: [      import devSeed from "@src/dbSeeds.js"    ],  }}
```

main.wasp

```
    app MyApp {  title: "My app",  // ...  db: {    seeds: [      import devSeed from "@src/dbSeeds.js"    ],  }}
```

`app.db` is a dictionary with the following fields (all fields are optional):

-   `seeds: [ExtImport]`
    
    Defines the seed functions you can use with the `wasp db seed` command to seed your database with initial data. Read the [Seeding section](#seeding-the-database) for more details.
    

### CLI Commands for Seeding the Database[​](#cli-commands-for-seeding-the-database "Direct link to CLI Commands for Seeding the Database")

Use one of the following commands to run the seed functions:

-   `wasp db seed`
    
    If you've only defined a single seed function, this command runs it. If you've defined multiple seed functions, it asks you to choose one interactively.
    
-   `wasp db seed <seed-name>`
    
    This command runs the seed function with the specified name. The name is the identifier used in its `import` expression in the `app.db.seeds` list. For example, to run the seed function `devSeedSimple` which was defined like this:
    
    -   JavaScript
    -   TypeScript
    
    main.wasp
    
    ```
        app MyApp {  // ...  db: {    seeds: [      // ...      import { devSeedSimple } from "@src/dbSeeds.js",    ]  }}
    ```
    
    main.wasp
    
    ```
        app MyApp {  // ...  db: {    seeds: [      // ...      import { devSeedSimple } from "@src/dbSeeds.js",    ]  }}
    ```
    
    Use the following command:
    
    ```
        wasp db seed devSeedSimple
    ```

---

# Prisma Schema File

Wasp uses [Prisma](https://www.prisma.io/) to interact with the database. Prisma is a "Next-generation Node.js and TypeScript ORM" that provides a type-safe API for working with your database.

With Prisma, you define your application's data model in a `schema.prisma` file. Read more about how Wasp Entities relate to Prisma models on the [Entities](/docs/data-model/entities) page.

In Wasp, the `schema.prisma` file is located in your project's root directory:

```
    .├── main.wasp...├── schema.prisma├── src├── tsconfig.json└── vite.config.ts
```

Wasp uses the `schema.prisma` file to understand your app's data model and generate the necessary code to interact with the database.

## Wasp file and Prisma schema file[​](#wasp-file-and-prisma-schema-file "Direct link to Wasp file and Prisma schema file")

Let's see how Wasp and Prisma files work together to define your application.

Here's an example `schema.prisma` file where we defined some database options and two models (User and Task) with a one-to-many relationship:

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}generator client {  provider = "prisma-client-js"}model User {  id      Int        @id @default(autoincrement())  tasks   Task[]}model Task {  id          Int        @id @default(autoincrement())  description String  isDone      Boolean    @default(false)  user        User       @relation(fields: [userId], references: [id])  userId      Int}
```

Wasp reads this `schema.prisma` file and extracts the info about your database models and database config.

The `datasource` block defines which database you want to use (PostgreSQL in this case) and some other options.

The `generator` block defines how to generate the Prisma Client code that you can use in your application to interact with the database.

![Relationship between Wasp file and Prisma file](/img/data-model/prisma_in_wasp.png)

Relationship between Wasp file and Prisma file

Finally, Prisma models become Wasp Entities which can be then used in the `main.wasp` file:

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",}...// Using Wasp Entities in the Wasp filequery getTasks {  fn: import { getTasks } from "@src/queries",  entities: [Task]}job myJob {  executor: PgBoss,  perform: {    fn: import { foo } from "@src/workers/bar"  },  entities: [Task],}api fooBar {  fn: import { fooBar } from "@src/apis",  entities: [Task],  httpRoute: (GET, "/foo/bar/:email")}
```

In the implementation of the `getTasks` query, `Task` is a Wasp Entity that corresponds to the `Task` model defined in the `schema.prisma` file.

The same goes for the `myJob` job and `fooBar` API, where `Task` is used as an Entity.

To learn more about the relationship between Wasp Entities and Prisma models, check out the [Entities](/docs/data-model/entities) page.

## Wasp-specific Prisma configuration[​](#wasp-specific-prisma-configuration "Direct link to Wasp-specific Prisma configuration")

Wasp mostly lets you use the Prisma schema file as you would in any other JS/TS project. However, there are some Wasp-specific rules you need to follow.

### The `datasource` block[​](#the-datasource-block "Direct link to the-datasource-block")

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}
```

Wasp takes the `datasource` you write and use it as-is.

There are some rules you need to follow:

-   You can only use `"postgresql"` or `"sqlite"` as the `provider` because Wasp only supports PostgreSQL and SQLite databases for now.
-   You must set the `url` field to `env("DATABASE_URL")` so that Wasp can work properly with your database.

### The `generator` blocks[​](#the-generator-blocks "Direct link to the-generator-blocks")

schema.prisma

```
    generator client {  provider = "prisma-client-js"}
```

Wasp requires that there is a `generator` block with `provider = "prisma-client-js"` in the `schema.prisma` file.

You can add additional generators if you need them in your project.

### The `model` blocks[​](#the-model-blocks "Direct link to the-model-blocks")

schema.prisma

```
    model User {  id      Int        @id @default(autoincrement())  tasks   Task[]}model Task {  id          Int        @id @default(autoincrement())  description String  isDone      Boolean    @default(false)  user        User       @relation(fields: [userId], references: [id])  userId      Int}
```

You can define your models in any way you like, if it's valid Prisma schema code, it will work with Wasp.

Triple slash comments

Wasp doesn't yet fully support `/// comment` syntax in the `schema.prisma` file. We are tracking it [here](https://github.com/wasp-lang/wasp/issues/2132), let us know if this is something you need.

## Prisma preview features[​](#prisma-preview-features "Direct link to Prisma preview features")

Prisma is still in active development and some of its features are not yet stable. To enable various preview features in Prisma, you need to add the `previewFeatures` field to the `generator` block in the `schema.prisma` file.

For example, one useful Prisma preview feature is PostgreSQL extensions support, which allows you to use PostgreSQL extensions like `pg_vector` or `pg_trgm` in your database schema:

schema.prisma

```
    datasource db {  provider   = "postgresql"  url        = env("DATABASE_URL")  extensions = [pgvector(map: "vector")]}generator client {  provider        = "prisma-client-js"  previewFeatures = ["postgresqlExtensions"]}// ...
```

Read more about preview features in the Prisma docs [here](https://www.prisma.io/docs/orm/reference/preview-features/client-preview-features) or about using PostgreSQL extensions [here](https://www.prisma.io/docs/orm/prisma-schema/postgresql-extensions).

---

# Overview

Auth is an essential piece of any serious application. That's why Wasp provides authentication and authorization support out of the box.

Here's a 1-minute tour of how full-stack auth works in Wasp:

Enabling auth for your app is optional and can be done by configuring the `auth` field of your `app` declaration:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  //...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {}, // use this or email, not both      email: {}, // use this or usernameAndPassword, not both      google: {},      gitHub: {},    },    onAuthFailedRedirectTo: "/someRoute"  }}//...
```

main.wasp

```
    app MyApp {  title: "My app",  //...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {}, // use this or email, not both      email: {}, // use this or usernameAndPassword, not both      google: {},      gitHub: {},    },    onAuthFailedRedirectTo: "/someRoute"  }}//...
```

Read more about the `auth` field options in the [API Reference](#api-reference) section.

We will provide a quick overview of auth in Wasp and link to more detailed documentation for each auth method.

## Available auth methods[​](#available-auth-methods "Direct link to Available auth methods")

Wasp supports the following auth methods:

[

### Email »

Email verification, password reset, etc.

](/docs/auth/email)[

### Username & Password »

The simplest way to get started

](/docs/auth/username-and-pass)[

### Google »

Users sign in with their Google account

](/docs/auth/social-auth/google)[

### Github »

Users sign in with their Github account

](/docs/auth/social-auth/github)[

### Keycloak »

Users sign in with their Keycloak account

](/docs/auth/social-auth/keycloak)[

### Discord »

Users sign in with their Discord account

](/docs/auth/social-auth/discord)

Click on each auth method for more details.

Let's say we enabled the [Username & password](/docs/auth/username-and-pass) authentication.

We get an auth backend with signup and login endpoints. We also get the `user` object in our [Operations](/docs/data-model/operations/overview) and we can decide what to do based on whether the user is logged in or not.

We would also get the [Auth UI](/docs/auth/ui) generated for us. We can set up our login and signup pages where our users can **create their account** and **login**. We can then protect certain pages by setting `authRequired: true` for them. This will make sure that only logged-in users can access them.

We will also have access to the `user` object in our frontend code, so we can show different UI to logged-in and logged-out users. For example, we can show the user's name in the header alongside a **logout button** or a login button if the user is not logged in.

## Protecting a page with `authRequired`[​](#protecting-a-page-with-authrequired "Direct link to protecting-a-page-with-authrequired")

When declaring a page, you can set the `authRequired` property.

If you set it to `true`, only authenticated users can access the page. Unauthenticated users are redirected to a route defined by the `app.auth.onAuthFailedRedirectTo` field.

-   JavaScript
-   TypeScript

main.wasp

```
    page MainPage {  component: import Main from "@src/pages/Main",  authRequired: true}
```

main.wasp

```
    page MainPage {  component: import Main from "@src/pages/Main",  authRequired: true}
```

Requires auth method

You can only use `authRequired` if your app uses one of the [available auth methods](#available-auth-methods).

If `authRequired` is set to `true`, the page's React component (specified by the `component` property) receives the `user` object as a prop. Read more about the `user` object in the [Accessing the logged-in user section](#accessing-the-logged-in-user).

## Logout action[​](#logout-action "Direct link to Logout action")

We provide an action for logging out the user. Here's how you can use it:

-   JavaScript
-   TypeScript

src/components/LogoutButton.jsx

```
    import { logout } from 'wasp/client/auth'const LogoutButton = () => {  return <button onClick={logout}>Logout</button>}
```

src/components/LogoutButton.tsx

```
    import { logout } from 'wasp/client/auth'const LogoutButton = () => {  return <button onClick={logout}>Logout</button>}
```

## Accessing the logged-in user[​](#accessing-the-logged-in-user "Direct link to Accessing the logged-in user")

You can get access to the `user` object both on the server and on the client. The `user` object contains the logged-in user's data.

The `user` object has all the fields that you defined in your `User` entity. In addition to that, it will also contain all the auth-related fields that Wasp stores. This includes things like the `username` or the email verification status. For example, if you have a user that signed up using an email and password, the `user` object might look like this:

```
    const user = {  // User data  id: "cluqsex9500017cn7i2hwsg17",  address: "Some address",  // Auth methods specific data  identities: {    email: {      id: "user@app.com",      isEmailVerified: true,      emailVerificationSentAt: "2024-04-08T10:06:02.204Z",      passwordResetSentAt: null,    },  },}
```

You can read more about how the `User` is connected to the rest of the auth system and how you can access the user data in the [Accessing User Data](/docs/auth/entities) section of the docs.

### On the client[​](#on-the-client "Direct link to On the client")

There are two ways to access the `user` object on the client:

-   the `user` prop
-   the `useAuth` hook

#### Using the `user` prop[​](#using-the-user-prop "Direct link to using-the-user-prop")

If the page's declaration sets `authRequired` to `true`, the page's React component receives the `user` object as a prop:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...page AccountPage {  component: import Account from "@src/pages/Account",  authRequired: true}
```

src/pages/Account.jsx

```
    import Button from './Button'import { logout } from 'wasp/client/auth'const AccountPage = ({ user }) => {  return (    <div>      <Button onClick={logout}>Logout</Button>      {JSON.stringify(user, null, 2)}    </div>  )}export default AccountPage
```

main.wasp

```
    // ...page AccountPage {  component: import Account from "@src/pages/Account",  authRequired: true}
```

src/pages/Account.tsx

```
    import { type AuthUser } from 'wasp/auth'import Button from './Button'import { logout } from 'wasp/client/auth'const AccountPage = ({ user }: { user: AuthUser }) => {  return (    <div>      <Button onClick={logout}>Logout</Button>      {JSON.stringify(user, null, 2)}    </div>  )}export default AccountPage
```

#### Using the `useAuth` hook[​](#using-the-useauth-hook "Direct link to using-the-useauth-hook")

Wasp provides a React hook you can use in the client components - `useAuth`.

This hook is a thin wrapper over Wasp's `useQuery` hook and returns data in the same format.

-   JavaScript
-   TypeScript

src/pages/MainPage.jsx

```
    import { useAuth, logout } from 'wasp/client/auth'import { Link } from 'react-router-dom'import Todo from '../Todo'export function Main() {  const { data: user } = useAuth()  if (!user) {    return (      <span>        Please <Link to="/login">login</Link> or{' '}        <Link to="/signup">sign up</Link>.      </span>    )  } else {    return (      <>        <button onClick={logout}>Logout</button>        <Todo />      </>    )  }}
```

src/pages/MainPage.tsx

```
    import { useAuth, logout } from 'wasp/client/auth'import { Link } from 'react-router-dom'import Todo from '../Todo'export function Main() {  const { data: user } = useAuth()  if (!user) {    return (      <span>        Please <Link to='/login'>login</Link> or <Link to='/signup'>sign up</Link>.      </span>    )  } else {    return (      <>        <button onClick={logout}>Logout</button>        <Todo />      < />    )  }}
```

tip

Since the `user` prop is only available in a page's React component: use the `user` prop in the page's React component and the `useAuth` hook in any other React component.

### On the server[​](#on-the-server "Direct link to On the server")

#### Using the `context.user` object[​](#using-the-contextuser-object "Direct link to using-the-contextuser-object")

When authentication is enabled, all [queries and actions](/docs/data-model/operations/overview) have access to the `user` object through the `context` argument. `context.user` contains all User entity's fields and the auth identities connected to the user. We strip out the `hashedPassword` field from the identities for security reasons.

-   JavaScript
-   TypeScript

src/actions.js

```
    import { HttpError } from 'wasp/server'export const createTask = async (task, context) => {  if (!context.user) {    throw new HttpError(403)  }  const Task = context.entities.Task  return Task.create({    data: {      description: task.description,      user: {        connect: { id: context.user.id },      },    },  })}
```

src/actions.ts

```
    import { type Task } from 'wasp/entities'import { type CreateTask } from 'wasp/server/operations'import { HttpError } from 'wasp/server'type CreateTaskPayload = Pick<Task, 'description'>export const createTask: CreateTask<CreateTaskPayload, Task> = async (  args,  context) => {  if (!context.user) {    throw new HttpError(403)  }  const Task = context.entities.Task  return Task.create({    data: {      description: args.description,      user: {        connect: { id: context.user.id },      },    },  })}
```

To implement access control in your app, each operation must check `context.user` and decide what to do. For example, if `context.user` is `undefined` inside a private operation, the user's access should be denied.

When using WebSockets, the `user` object is also available on the `socket.data` object. Read more in the [WebSockets section](/docs/advanced/web-sockets#websocketfn-function).

## Sessions[​](#sessions "Direct link to Sessions")

Wasp's auth uses sessions to keep track of the logged-in user. The session is stored in `localStorage` on the client and in the database on the server. Under the hood, Wasp uses the excellent [Lucia Auth v3](https://v3.lucia-auth.com/) library for session management.

When users log in, Wasp creates a session for them and stores it in the database. The session is then sent to the client and stored in `localStorage`. When users log out, Wasp deletes the session from the database and from `localStorage`.

## User Entity[​](#user-entity "Direct link to User Entity")

### Password Hashing[​](#password-hashing "Direct link to Password Hashing")

If you are saving a user's password in the database, you should **never** save it as plain text. You can use Wasp's helper functions for serializing and deserializing provider data which will automatically hash the password for you:

main.wasp

```
    // ...action updatePassword {  fn: import { updatePassword } from "@src/auth",}
```

-   JavaScript
-   TypeScript

src/auth.js

```
    import {  createProviderId,  findAuthIdentity,  updateAuthIdentityProviderData,  deserializeAndSanitizeProviderData,} from 'wasp/server/auth';export const updatePassword = async (args, context) => {  const providerId = createProviderId('email', args.email)  const authIdentity = await findAuthIdentity(providerId)  if (!authIdentity) {      throw new HttpError(400, "Unknown user")  }    const providerData = deserializeAndSanitizeProviderData(authIdentity.providerData)  // Updates the password and hashes it automatically.  await updateAuthIdentityProviderData(providerId, providerData, {      hashedPassword: args.password,  })}
```

src/auth.ts

```
    import {  createProviderId,  findAuthIdentity,  updateAuthIdentityProviderData,  deserializeAndSanitizeProviderData,} from 'wasp/server/auth';import { type UpdatePassword } from 'wasp/server/operations'export const updatePassword: UpdatePassword<  { email: string; password: string },  void,> = async (args, context) => {  const providerId = createProviderId('email', args.email)  const authIdentity = await findAuthIdentity(providerId)  if (!authIdentity) {      throw new HttpError(400, "Unknown user")  }    const providerData = deserializeAndSanitizeProviderData<'email'>(authIdentity.providerData)  // Updates the password and hashes it automatically.  await updateAuthIdentityProviderData(providerId, providerData, {      hashedPassword: args.password,  })}
```

### Default Validations[​](#default-validations "Direct link to Default Validations")

When you are using the default authentication flow, Wasp validates the fields with some default validations. These validations run if you use Wasp's built-in [Auth UI](/docs/auth/ui) or if you use the provided auth actions.

If you decide to create your [custom auth actions](/docs/auth/username-and-pass#2-creating-your-custom-sign-up-action), you'll need to run the validations yourself.

Default validations depend on the auth method you use.

#### Username & Password[​](#username--password "Direct link to Username & Password")

If you use [Username & password](/docs/auth/username-and-pass) authentication, the default validations are:

-   The `username` must not be empty
-   The `password` must not be empty, have at least 8 characters, and contain a number

Note that `username`s are stored in a **case-insensitive** manner.

#### Email[​](#email "Direct link to Email")

If you use [Email](/docs/auth/email) authentication, the default validations are:

-   The `email` must not be empty and a valid email address
-   The `password` must not be empty, have at least 8 characters, and contain a number

Note that `email`s are stored in a **case-insensitive** manner.

## Customizing the Signup Process[​](#customizing-the-signup-process "Direct link to Customizing the Signup Process")

Sometimes you want to include **extra fields** in your signup process, like first name and last name and save them in the `User` entity.

For this to happen:

-   you need to define the fields that you want saved in the database,
-   you need to customize the `SignupForm` (in the case of [Email](/docs/auth/email) or [Username & Password](/docs/auth/username-and-pass) auth)

Other times, you might need to just add some **extra UI** elements to the form, like a checkbox for terms of service. In this case, customizing only the UI components is enough.

Let's see how to do both.

### 1\. Defining Extra Fields[​](#1-defining-extra-fields "Direct link to 1. Defining Extra Fields")

If we want to **save** some extra fields in our signup process, we need to tell our app they exist.

We do that by defining an object where the keys represent the field name, and the values are functions that receive the data sent from the client\* and return the value of the field.

\* We exclude the `password` field from this object to prevent it from being saved as plain-text in the database. The `password` field is handled by Wasp's auth backend.

First, we add the `auth.methods.{authMethod}.userSignupFields` field in our `main.wasp` file. The `{authMethod}` depends on the auth method you are using.

For example, if you are using [Username & Password](/docs/auth/username-and-pass), you would add the `auth.methods.usernameAndPassword.userSignupFields` field:

-   JavaScript
-   TypeScript

main.wasp

```
    app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {        userSignupFields: import { userSignupFields } from "@src/auth/signup",      },    },    onAuthFailedRedirectTo: "/login",  },}
```

schema.prisma

```
    model User {  id      Int     @id @default(autoincrement())  address String?}
```

Then we'll define the `userSignupFields` object in the `src/auth/signup.js` file:

src/auth/signup.js

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: async (data) => {    const address = data.address    if (typeof address !== 'string') {      throw new Error('Address is required')    }    if (address.length < 5) {      throw new Error('Address must be at least 5 characters long')    }    return address  },})
```

main.wasp

```
    app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {        userSignupFields: import { userSignupFields } from "@src/auth/signup",      },    },    onAuthFailedRedirectTo: "/login",  },}
```

schema.prisma

```
    model User {  id      Int     @id @default(autoincrement())  address String?}
```

Then we'll define the `userSignupFields` object in the `src/auth/signup.js` file:

src/auth/signup.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: async (data) => {    const address = data.address    if (typeof address !== 'string') {      throw new Error('Address is required')    }    if (address.length < 5) {      throw new Error('Address must be at least 5 characters long')    }    return address  },})
```

Read more about the `userSignupFields` object in the [API Reference](#signup-fields-customization).

Keep in mind, that these field names need to exist on the `userEntity` you defined in your `main.wasp` file e.g. `address` needs to be a field on the `User` entity you defined in the `schema.prisma` file.

The field function will receive the data sent from the client and it needs to return the value that will be saved into the database. If the field is invalid, the function should throw an error.

Using Validation Libraries

You can use any validation library you want to validate the fields. For example, you can use `zod` like this:

Click to see the code

-   JavaScript
-   TypeScript

src/auth/signup.js

```
    import { defineUserSignupFields } from 'wasp/server/auth'import * as z from 'zod'export const userSignupFields = defineUserSignupFields({  address: (data) => {    const AddressSchema = z      .string({        required_error: 'Address is required',        invalid_type_error: 'Address must be a string',      })      .min(10, 'Address must be at least 10 characters long')    const result = AddressSchema.safeParse(data.address)    if (result.success === false) {      throw new Error(result.error.issues[0].message)    }    return result.data  },})
```

src/auth/signup.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'import * as z from 'zod'export const userSignupFields = defineUserSignupFields({  address: (data) => {    const AddressSchema = z      .string({        required_error: 'Address is required',        invalid_type_error: 'Address must be a string',      })      .min(10, 'Address must be at least 10 characters long')    const result = AddressSchema.safeParse(data.address)    if (result.success === false) {      throw new Error(result.error.issues[0].message)    }    return result.data  },})
```

Now that we defined the fields, Wasp knows how to:

1.  Validate the data sent from the client
2.  Save the data to the database

Next, let's see how to customize [Auth UI](/docs/auth/ui) to include those fields.

### 2\. Customizing the Signup Component[​](#2-customizing-the-signup-component "Direct link to 2. Customizing the Signup Component")

Using Custom Signup Component

If you are not using Wasp's Auth UI, you can skip this section. Just make sure to include the extra fields in your custom signup form.

Read more about using the signup actions for:

-   email auth [here](/docs/auth/email#fields-in-the-email-dict)
-   username & password auth [here](/docs/auth/username-and-pass#customizing-the-auth-flow)

If you are using Wasp's Auth UI, you can customize the `SignupForm` component by passing the `additionalFields` prop to it. It can be either a list of extra fields or a render function.

#### Using a List of Extra Fields[​](#using-a-list-of-extra-fields "Direct link to Using a List of Extra Fields")

When you pass in a list of extra fields to the `SignupForm`, they are added to the form one by one, in the order you pass them in.

Inside the list, there can be either **objects** or **render functions** (you can combine them):

1.  Objects are a simple way to describe new fields you need, but a bit less flexible than render functions.
2.  Render functions can be used to render any UI you want, but they require a bit more code. The render functions receive the `react-hook-form` object and the form state object as arguments.

-   JavaScript
-   TypeScript

src/SignupPage.jsx

```
    import {  SignupForm,  FormError,  FormInput,  FormItemGroup,  FormLabel,} from 'wasp/client/auth'export const SignupPage = () => {  return (    <SignupForm      additionalFields={[        /* The address field is defined using an object */        {          name: 'address',          label: 'Address',          type: 'input',          validations: {            required: 'Address is required',          },        },        /* The phone number is defined using a render function */        (form, state) => {          return (            <FormItemGroup>              <FormLabel>Phone Number</FormLabel>              <FormInput                {...form.register('phoneNumber', {                  required: 'Phone number is required',                })}                disabled={state.isLoading}              />              {form.formState.errors.phoneNumber && (                <FormError>                  {form.formState.errors.phoneNumber.message}                </FormError>              )}            </FormItemGroup>          )        },      ]}    />  )}
```

src/SignupPage.tsx

```
    import {  SignupForm,  FormError,  FormInput,  FormItemGroup,  FormLabel,} from 'wasp/client/auth'export const SignupPage = () => {  return (    <SignupForm      additionalFields={[        /* The address field is defined using an object */        {          name: 'address',          label: 'Address',          type: 'input',          validations: {            required: 'Address is required',          },        },        /* The phone number is defined using a render function */        (form, state) => {          return (            <FormItemGroup>              <FormLabel>Phone Number</FormLabel>              <FormInput                {...form.register('phoneNumber', {                  required: 'Phone number is required',                })}                disabled={state.isLoading}              />              {form.formState.errors.phoneNumber && (                <FormError>                  {form.formState.errors.phoneNumber.message}                </FormError>              )}            </FormItemGroup>          )        },      ]}    />  )}
```

Read more about the extra fields in the [API Reference](#signupform-customization).

#### Using a Single Render Function[​](#using-a-single-render-function "Direct link to Using a Single Render Function")

Instead of passing in a list of extra fields, you can pass in a render function which will receive the `react-hook-form` object and the form state object as arguments. What ever the render function returns, will be rendered below the default fields.

-   JavaScript
-   TypeScript

src/SignupPage.jsx

```
    import { SignupForm, FormItemGroup } from 'wasp/client/auth'export const SignupPage = () => {  return (    <SignupForm      additionalFields={(form, state) => {        const username = form.watch('username')        return (          username && (            <FormItemGroup>              Hello there <strong>{username}</strong> 👋            </FormItemGroup>          )        )      }}    />  )}
```

src/SignupPage.tsx

```
    import { SignupForm, FormItemGroup } from 'wasp/client/auth'export const SignupPage = () => {  return (    <SignupForm      additionalFields={(form, state) => {        const username = form.watch('username')        return (          username && (            <FormItemGroup>              Hello there <strong>{username}</strong> 👋            </FormItemGroup>          )        )      }}    />  )}
```

Read more about the render function in the [API Reference](#signupform-customization).

## API Reference[​](#api-reference "Direct link to API Reference")

### Auth Fields[​](#auth-fields "Direct link to Auth Fields")

-   JavaScript
-   TypeScript

main.wasp

```
      title: "My app",  //...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {}, // use this or email, not both      email: {}, // use this or usernameAndPassword, not both      google: {},      gitHub: {},    },    onAuthFailedRedirectTo: "/someRoute",  }}//...
```

main.wasp

```
    app MyApp {  title: "My app",  //...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {}, // use this or email, not both      email: {}, // use this or usernameAndPassword, not both      google: {},      gitHub: {},    },    onAuthFailedRedirectTo: "/someRoute",  }}//...
```

`app.auth` is a dictionary with the following fields:

#### `userEntity: entity` required[​](#userentity-entity- "Direct link to userentity-entity-")

The entity representing the user connected to your business logic.

You can read more about how the `User` is connected to the rest of the auth system and how you can access the user data in the [Accessing User Data](/docs/auth/entities) section of the docs.

#### `methods: dict` required[​](#methods-dict- "Direct link to methods-dict-")

A dictionary of auth methods enabled for the app.

[

### Email »

Email verification, password reset, etc.

](/docs/auth/email)[

### Username & Password »

The simplest way to get started

](/docs/auth/username-and-pass)[

### Google »

Users sign in with their Google account

](/docs/auth/social-auth/google)[

### Github »

Users sign in with their Github account

](/docs/auth/social-auth/github)[

### Keycloak »

Users sign in with their Keycloak account

](/docs/auth/social-auth/keycloak)[

### Discord »

Users sign in with their Discord account

](/docs/auth/social-auth/discord)

Click on each auth method for more details.

#### `onAuthFailedRedirectTo: String` required[​](#onauthfailedredirectto-string- "Direct link to onauthfailedredirectto-string-")

The route to which Wasp should redirect unauthenticated user when they try to access a private page (i.e., a page that has `authRequired: true`). Check out these [essential docs on auth](/docs/tutorial/auth#adding-auth-to-the-project) to see an example of usage.

#### `onAuthSucceededRedirectTo: String`[​](#onauthsucceededredirectto-string "Direct link to onauthsucceededredirectto-string")

The route to which Wasp will send a successfully authenticated after a successful login/signup. The default value is `"/"`.

note

Automatic redirect on successful login only works when using the Wasp-provided [Auth UI](/docs/auth/ui).

### Signup Fields Customization[​](#signup-fields-customization "Direct link to Signup Fields Customization")

If you want to add extra fields to the signup process, the server needs to know how to save them to the database. You do that by defining the `auth.methods.{authMethod}.userSignupFields` field in your `main.wasp` file.

-   JavaScript
-   TypeScript

main.wasp

```
    app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {        userSignupFields: import { userSignupFields } from "@src/auth/signup",      },    },    onAuthFailedRedirectTo: "/login",  },}
```

Then we'll export the `userSignupFields` object from the `src/auth/signup.js` file:

src/auth/signup.js

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: async (data) => {    const address = data.address    if (typeof address !== 'string') {      throw new Error('Address is required')    }    if (address.length < 5) {      throw new Error('Address must be at least 5 characters long')    }    return address  },})
```

main.wasp

```
    app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      usernameAndPassword: {        userSignupFields: import { userSignupFields } from "@src/auth/signup",      },    },    onAuthFailedRedirectTo: "/login",  },}
```

Then we'll export the `userSignupFields` object from the `src/auth/signup.ts` file:

src/auth/signup.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: async (data) => {    const address = data.address    if (typeof address !== 'string') {      throw new Error('Address is required')    }    if (address.length < 5) {      throw new Error('Address must be at least 5 characters long')    }    return address  },})
```

The `userSignupFields` object is an object where the keys represent the field name, and the values are functions that receive the data sent from the client\* and return the value of the field.

If the value that the function received is invalid, the function should throw an error.

\* We exclude the `password` field from this object to prevent it from being saved as plain text in the database. The `password` field is handled by Wasp's auth backend.

### `SignupForm` Customization[​](#signupform-customization "Direct link to signupform-customization")

To customize the `SignupForm` component, you need to pass in the `additionalFields` prop. It can be either a list of extra fields or a render function.

-   JavaScript
-   TypeScript

src/SignupPage.jsx

```
    import {  SignupForm,  FormError,  FormInput,  FormItemGroup,  FormLabel,} from 'wasp/client/auth'export const SignupPage = () => {  return (    <SignupForm      additionalFields={[        {          name: 'address',          label: 'Address',          type: 'input',          validations: {            required: 'Address is required',          },        },        (form, state) => {          return (            <FormItemGroup>              <FormLabel>Phone Number</FormLabel>              <FormInput                {...form.register('phoneNumber', {                  required: 'Phone number is required',                })}                disabled={state.isLoading}              />              {form.formState.errors.phoneNumber && (                <FormError>                  {form.formState.errors.phoneNumber.message}                </FormError>              )}            </FormItemGroup>          )        },      ]}    />  )}
```

src/SignupPage.tsx

```
    import {  SignupForm,  FormError,  FormInput,  FormItemGroup,  FormLabel,} from 'wasp/client/auth'export const SignupPage = () => {  return (    <SignupForm      additionalFields={[        {          name: 'address',          label: 'Address',          type: 'input',          validations: {            required: 'Address is required',          },        },        (form, state) => {          return (            <FormItemGroup>              <FormLabel>Phone Number</FormLabel>              <FormInput                {...form.register('phoneNumber', {                  required: 'Phone number is required',                })}                disabled={state.isLoading}              />              {form.formState.errors.phoneNumber && (                <FormError>                  {form.formState.errors.phoneNumber.message}                </FormError>              )}            </FormItemGroup>          )        },      ]}    />  )}
```

The extra fields can be either **objects** or **render functions** (you can combine them):

1.  Objects are a simple way to describe new fields you need, but a bit less flexible than render functions.
    
    The objects have the following properties:
    
    -   `name` required
        
        -   the name of the field
    -   `label` required
        
        -   the label of the field (used in the UI)
    -   `type` required
        
        -   the type of the field, which can be `input` or `textarea`
    -   `validations`
        
        -   an object with the validation rules for the field. The keys are the validation names, and the values are the validation error messages. Read more about the available validation rules in the [react-hook-form docs](https://react-hook-form.com/api/useform/register#register).
2.  Render functions receive the `react-hook-form` object and the form state as arguments, and they can use them to render arbitrary UI elements.
    
    The render function has the following signature:
    
    ```
        (form: UseFormReturn, state: FormState) => React.ReactNode
    ```
    
    -   `form` required
        
        -   the `react-hook-form` object, read more about it in the [react-hook-form docs](https://react-hook-form.com/api/useform)
        -   you need to use the `form.register` function to register your fields
    -   `state` required
        
        -   the form state object which has the following properties:
            -   `isLoading: boolean`
                -   whether the form is currently submitting

---

# Auth UI

To make using authentication in your app as easy as possible, Wasp generates the server-side code but also the client-side UI for you. It enables you to quickly get the login, signup, password reset and email verification flows in your app.

Below we cover all of the available UI components and how to use them.

![Auth UI](/assets/images/all_screens-4474bfaf188590d975e11d5b4e04b0ae.gif)

## Overview[​](#overview "Direct link to Overview")

After Wasp generates the UI components for your auth, you can use it as is, or customize it to your liking.

Based on the authentication providers you enabled in your `main.wasp` file, the Auth UI will show the corresponding UI (form and buttons). For example, if you enabled e-mail authentication:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  //...  auth: {    methods: {      email: {},    },    // ...  }}
```

main.wasp

```
    app MyApp {  //...  auth: {    methods: {      email: {},    },    // ...  }}
```

You'll get the following UI:

![Auth UI](/assets/images/login-1dad984cebbef0e4bd2bc0c008d2d2ff.png)

And then if you enable Google and Github:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  //...  auth: {    methods: {      email: {},      google: {},      github: {},    },    // ...  }}
```

main.wasp

```
    app MyApp {  //...  auth: {    methods: {      email: {},      google: {},      github: {},    },    // ...  }}
```

The form will automatically update to look like this:

![Auth UI](/assets/images/multiple_providers-68bde90e9254bf20b979986b6fd37c13.png)

Let's go through all of the available components and how to use them.

## Auth Components[​](#auth-components "Direct link to Auth Components")

The following components are available for you to use in your app:

-   [Login form](#login-form)
-   [Signup form](#signup-form)
-   [Forgot password form](#forgot-password-form)
-   [Reset password form](#reset-password-form)
-   [Verify email form](#verify-email-form)

### Login Form[​](#login-form "Direct link to Login Form")

Used with [Username & Password](/docs/auth/username-and-pass), [Email](/docs/auth/email), [Github](/docs/auth/social-auth/github), [Google](/docs/auth/social-auth/google), [Keycloak](/docs/auth/social-auth/keycloak), and [Discord](/docs/auth/social-auth/discord) authentication.

![Login form](/assets/images/login-1dad984cebbef0e4bd2bc0c008d2d2ff.png)

You can use the `LoginForm` component to build your login page:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { LoginPage } from "@src/LoginPage.jsx"}
```

src/LoginPage.jsx

```
    import { LoginForm } from 'wasp/client/auth'// Use it like thisexport function LoginPage() {  return <LoginForm />}
```

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { LoginPage } from "@src/LoginPage.tsx"}
```

src/LoginPage.tsx

```
    import { LoginForm } from 'wasp/client/auth'// Use it like thisexport function LoginPage() {  return <LoginForm />}
```

It will automatically show the correct authentication providers based on your `main.wasp` file.

### Signup Form[​](#signup-form "Direct link to Signup Form")

Used with [Username & Password](/docs/auth/username-and-pass), [Email](/docs/auth/email), [Github](/docs/auth/social-auth/github), [Google](/docs/auth/social-auth/google), [Keycloak](/docs/auth/social-auth/keycloak), and [Discord](/docs/auth/social-auth/discord) authentication.

![Signup form](/assets/images/signup-4362e29083e15457eb5a16990f57819a.png)

You can use the `SignupForm` component to build your signup page:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { SignupPage } from "@src/SignupPage.jsx"}
```

src/SignupPage.jsx

```
    import { SignupForm } from 'wasp/client/auth'// Use it like thisexport function SignupPage() {  return <SignupForm />}
```

main.wasp

```
    // ...route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { SignupPage } from "@src/SignupPage.tsx"}
```

src/SignupPage.tsx

```
    import { SignupForm } from 'wasp/client/auth'// Use it like thisexport function SignupPage() {  return <SignupForm />}
```

It will automatically show the correct authentication providers based on your `main.wasp` file.

Read more about customizing the signup process like adding additional fields or extra UI in the [Auth Overview](/docs/auth/overview#customizing-the-signup-process) section.

### Forgot Password Form[​](#forgot-password-form "Direct link to Forgot Password Form")

Used with [Email](/docs/auth/email) authentication.

If users forget their password, they can use this form to reset it.

![Forgot password form](/assets/images/forgot_password-493aa5ed988f04edef5cf4d400d51b95.png)

You can use the `ForgotPasswordForm` component to build your own forgot password page:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route RequestPasswordResetRoute { path: "/request-password-reset", to: RequestPasswordResetPage }page RequestPasswordResetPage {  component: import { ForgotPasswordPage } from "@src/ForgotPasswordPage.jsx"}
```

src/ForgotPasswordPage.jsx

```
    import { ForgotPasswordForm } from 'wasp/client/auth'// Use it like thisexport function ForgotPasswordPage() {  return <ForgotPasswordForm />}
```

main.wasp

```
    // ...route RequestPasswordResetRoute { path: "/request-password-reset", to: RequestPasswordResetPage }page RequestPasswordResetPage {  component: import { ForgotPasswordPage } from "@src/ForgotPasswordPage.tsx"}
```

src/ForgotPasswordPage.tsx

```
    import { ForgotPasswordForm } from 'wasp/client/auth'// Use it like thisexport function ForgotPasswordPage() {  return <ForgotPasswordForm />}
```

### Reset Password Form[​](#reset-password-form "Direct link to Reset Password Form")

Used with [Email](/docs/auth/email) authentication.

After users click on the link in the email they receive after submitting the forgot password form, they will be redirected to this form where they can reset their password.

![Reset password form](/assets/images/reset_password-fe01997060cbe04a86c134098b59df46.png)

You can use the `ResetPasswordForm` component to build your reset password page:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route PasswordResetRoute { path: "/password-reset", to: PasswordResetPage }page PasswordResetPage {  component: import { ResetPasswordPage } from "@src/ResetPasswordPage.jsx"}
```

src/ResetPasswordPage.jsx

```
    import { ResetPasswordForm } from 'wasp/client/auth'// Use it like thisexport function ResetPasswordPage() {  return <ResetPasswordForm />}
```

main.wasp

```
    // ...route PasswordResetRoute { path: "/password-reset", to: PasswordResetPage }page PasswordResetPage {  component: import { ResetPasswordPage } from "@src/ResetPasswordPage.tsx"}
```

src/ResetPasswordPage.tsx

```
    import { ResetPasswordForm } from 'wasp/client/auth'// Use it like thisexport function ResetPasswordPage() {  return <ResetPasswordForm />}
```

### Verify Email Form[​](#verify-email-form "Direct link to Verify Email Form")

Used with [Email](/docs/auth/email) authentication.

After users sign up, they will receive an email with a link to this form where they can verify their email.

![Verify email form](/assets/images/email_verification-b1919c242edc678175636485f6fc2264.png)

You can use the `VerifyEmailForm` component to build your email verification page:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route EmailVerificationRoute { path: "/email-verification", to: EmailVerificationPage }page EmailVerificationPage {  component: import { VerifyEmailPage } from "@src/VerifyEmailPage.jsx"}
```

src/VerifyEmailPage.jsx

```
    import { VerifyEmailForm } from 'wasp/client/auth'// Use it like thisexport function VerifyEmailPage() {  return <VerifyEmailForm />}
```

main.wasp

```
    // ...route EmailVerificationRoute { path: "/email-verification", to: EmailVerificationPage }page EmailVerificationPage {  component: import { VerifyEmailPage } from "@src/VerifyEmailPage.tsx"}
```

src/VerifyEmailPage.tsx

```
    import { VerifyEmailForm } from 'wasp/client/auth'// Use it like thisexport function VerifyEmailPage() {  return <VerifyEmailForm />}
```

## Customization 💅🏻[​](#customization- "Direct link to Customization 💅🏻")

You customize all of the available forms by passing props to them.

Props you can pass to all of the forms:

1.  `appearance` - customize the form colors (via design tokens)
2.  `logo` - path to your logo
3.  `socialLayout` - layout of the social buttons, which can be `vertical` or `horizontal`

### 1\. Customizing the Colors[​](#1-customizing-the-colors "Direct link to 1. Customizing the Colors")

We use [Stitches](https://stitches.dev/) to style the Auth UI. You can customize the styles by overriding the default theme tokens.

List of all available tokens

See the [list of all available tokens](https://github.com/wasp-lang/wasp/blob/release/waspc/data/Generator/templates/react-app/src/stitches.config.js) which you can override.

-   JavaScript
-   TypeScript

src/appearance.js

```
    export const authAppearance = {  colors: {    brand: '#5969b8', // blue    brandAccent: '#de5998', // pink    submitButtonText: 'white',  },}
```

src/LoginPage.jsx

```
    import { LoginForm } from 'wasp/client/auth'import { authAppearance } from './appearance'export function LoginPage() {  return (    <LoginForm      // Pass the appearance object to the form      appearance={authAppearance}    />  )}
```

src/appearance.ts

```
    import type { CustomizationOptions } from 'wasp/client/auth'export const authAppearance: CustomizationOptions['appearance'] = {  colors: {    brand: '#5969b8', // blue    brandAccent: '#de5998', // pink    submitButtonText: 'white',  },}
```

src/LoginPage.tsx

```
    import { LoginForm } from 'wasp/client/auth'import { authAppearance } from './appearance'export function LoginPage() {  return (    <LoginForm      // Pass the appearance object to the form      appearance={authAppearance}    />  )}
```

We recommend defining your appearance in a separate file and importing it into your components.

### 2\. Using Your Logo[​](#2-using-your-logo "Direct link to 2. Using Your Logo")

You can add your logo to the Auth UI by passing the `logo` prop to any of the components.

-   JavaScript
-   TypeScript

src/LoginPage.jsx

```
    import { LoginForm } from 'wasp/client/auth'import Logo from './logo.png'export function LoginPage() {  return (    <LoginForm      // Pass in the path to your logo      logo={Logo}    />  )}
```

src/LoginPage.tsx

```
    import { LoginForm } from 'wasp/client/auth'import Logo from './logo.png'export function LoginPage() {  return (    <LoginForm      // Pass in the path to your logo      logo={Logo}    />  )}
```

### 3\. Social Buttons Layout[​](#3-social-buttons-layout "Direct link to 3. Social Buttons Layout")

You can change the layout of the social buttons by passing the `socialLayout` prop to any of the components. It can be either `vertical` or `horizontal` (default).

If we pass in `vertical`:

-   JavaScript
-   TypeScript

src/LoginPage.jsx

```
    import { LoginForm } from 'wasp/client/auth'export function LoginPage() {  return (    <LoginForm      // Pass in the socialLayout prop      socialLayout="vertical"    />  )}
```

src/LoginPage.tsx

```
    import { LoginForm } from 'wasp/client/auth'export function LoginPage() {  return (    <LoginForm      // Pass in the socialLayout prop      socialLayout="vertical"    />  )}
```

We get this:

![Vertical social buttons](/assets/images/vertical_social_buttons-98a5bea6f132202b7a94711a661267c0.png)

### Let's Put Everything Together 🪄[​](#lets-put-everything-together- "Direct link to Let's Put Everything Together 🪄")

If we provide the logo and custom colors:

-   JavaScript
-   TypeScript

src/appearance.js

```
    export const appearance = {  colors: {    brand: '#5969b8', // blue    brandAccent: '#de5998', // pink    submitButtonText: 'white',  },}
```

src/LoginPage.jsx

```
    import { LoginForm } from 'wasp/client/auth'import { authAppearance } from './appearance'import todoLogo from './todoLogo.png'export function LoginPage() {  return <LoginForm appearance={appearance} logo={todoLogo} />}
```

src/appearance.ts

```
    import type { CustomizationOptions } from 'wasp/client/auth'export const appearance: CustomizationOptions['appearance'] = {  colors: {    brand: '#5969b8', // blue    brandAccent: '#de5998', // pink    submitButtonText: 'white',  },}
```

src/LoginPage.tsx

```
    import { LoginForm } from 'wasp/client/auth'import { authAppearance } from './appearance'import todoLogo from './todoLogo.png'export function LoginPage() {  return <LoginForm appearance={appearance} logo={todoLogo} />}
```

We get a form looking like this:

![Custom login form](/img/authui/custom_login.gif)

---

# Username & Password

Wasp supports username & password authentication out of the box with login and signup flows. It provides you with the server-side implementation and the UI components for the client side.

## Setting Up Username & Password Authentication[​](#setting-up-username--password-authentication "Direct link to Setting Up Username & Password Authentication")

To set up username authentication we need to:

1.  Enable username authentication in the Wasp file
2.  Add the `User` entity
3.  Add the auth routes and pages
4.  Use Auth UI components in our pages

Structure of the `main.wasp` file we will end up with:

main.wasp

```
    // Configuring e-mail authenticationapp myApp {  auth: { ... }}// Defining routes and pagesroute SignupRoute { ... }page SignupPage { ... }// ...
```

### 1\. Enable Username Authentication[​](#1-enable-username-authentication "Direct link to 1. Enable Username Authentication")

Let's start with adding the following to our `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the user entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable username authentication      usernameAndPassword: {},    },    onAuthFailedRedirectTo: "/login"  }}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the user entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable username authentication      usernameAndPassword: {},    },    onAuthFailedRedirectTo: "/login"  }}
```

Read more about the `usernameAndPassword` auth method options [here](#fields-in-the-usernameandpassword-dict).

### 2\. Add the User Entity[​](#2-add-the-user-entity "Direct link to 2. Add the User Entity")

The `User` entity can be as simple as including only the `id` field:

-   JavaScript
-   TypeScript

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

You can read more about how the `User` is connected to the rest of the auth system and how you can access the user data in the [Accessing User Data](/docs/auth/entities) section of the docs.

### 3\. Add the Routes and Pages[​](#3-add-the-routes-and-pages "Direct link to 3. Add the Routes and Pages")

Next, we need to define the routes and pages for the authentication pages.

Add the following to the `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.jsx"}route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { Signup } from "@src/pages/auth.jsx"}
```

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.tsx"}route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { Signup } from "@src/pages/auth.tsx"}
```

We'll define the React components for these pages in the `src/pages/auth.tsx` file below.

### 4\. Create the Client Pages[​](#4-create-the-client-pages "Direct link to 4. Create the Client Pages")

info

We are using [Tailwind CSS](https://tailwindcss.com/) to style the pages. Read more about how to add it [here](/docs/project/css-frameworks).

Let's create a `auth.tsx` file in the `src/pages` folder and add the following to it:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import { LoginForm, SignupForm } from 'wasp/client/auth'import { Link } from 'react-router-dom'export function Login() {  return (    <Layout>      <LoginForm />      <br />      <span className="text-sm font-medium text-gray-900">        Don't have an account yet? <Link to="/signup">go to signup</Link>.      </span>    </Layout>  );}export function Signup() {  return (    <Layout>      <SignupForm />      <br />      <span className="text-sm font-medium text-gray-900">        I already have an account (<Link to="/login">go to login</Link>).      </span>    </Layout>  );}// A layout component to center the contentexport function Layout({ children }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  );}
```

src/pages/auth.tsx

```
    import { LoginForm, SignupForm } from 'wasp/client/auth'import { Link } from 'react-router-dom'export function Login() {  return (    <Layout>      <LoginForm />      <br />      <span className="text-sm font-medium text-gray-900">        Don't have an account yet? <Link to="/signup">go to signup</Link>.      </span>    </Layout>  );}export function Signup() {  return (    <Layout>      <SignupForm />      <br />      <span className="text-sm font-medium text-gray-900">        I already have an account (<Link to="/login">go to login</Link>).      </span>    </Layout>  );}// A layout component to center the contentexport function Layout({ children }: { children: React.ReactNode }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  );}
```

We imported the generated Auth UI components and used them in our pages. Read more about the Auth UI components [here](/docs/auth/ui).

### Conclusion[​](#conclusion "Direct link to Conclusion")

That's it! We have set up username authentication in our app. 🎉

Running `wasp db migrate-dev` and then `wasp start` should give you a working app with username authentication. If you want to put some of the pages behind authentication, read the [auth overview docs](/docs/auth/overview).

Using multiple auth identities for a single user

Wasp currently doesn't support multiple auth identities for a single user. This means, for example, that a user can't have both an email-based auth identity and a Google-based auth identity. This is something we will add in the future with the introduction of the [account merging feature](https://github.com/wasp-lang/wasp/issues/954).

Account merging means that multiple auth identities can be merged into a single user account. For example, a user's email and Google identity can be merged into a single user account. Then the user can log in with either their email or Google account and they will be logged into the same account.

## Customizing the Auth Flow[​](#customizing-the-auth-flow "Direct link to Customizing the Auth Flow")

The login and signup flows are pretty standard: they allow the user to sign up and then log in with their username and password. The signup flow validates the username and password and then creates a new user entity in the database.

Read more about the default username and password validation rules in the [auth overview docs](/docs/auth/overview#default-validations).

If you require more control in your authentication flow, you can achieve that in the following ways:

1.  Create your UI and use `signup` and `login` actions.
2.  Create your custom sign-up action which uses the lower-level API, along with your custom code.

### 1\. Using the `signup` and `login` actions[​](#1-using-the-signup-and-login-actions "Direct link to 1-using-the-signup-and-login-actions")

#### `login()`[​](#login "Direct link to login")

An action for logging in the user.

It takes two arguments:

-   `username: string` required
    
    Username of the user logging in.
    
-   `password: string` required
    
    Password of the user logging in.
    

You can use it like this:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import { login } from 'wasp/client/auth'import { useState } from 'react'import { useHistory, Link } from 'react-router-dom'export function LoginPage() {  const [username, setUsername] = useState('')  const [password, setPassword] = useState('')  const [error, setError] = useState(null)  const history = useHistory()  async function handleSubmit(event) {    event.preventDefault()    try {      await login(username, password)      history.push('/')    } catch (error) {      setError(error)    }  }  return (    <form onSubmit={handleSubmit}>      {/* ... */}    </form>  );}
```

src/pages/auth.tsx

```
    import { login } from 'wasp/client/auth'import { useState } from 'react'import { useHistory, Link } from 'react-router-dom'export function LoginPage() {  const [username, setUsername] = useState('')  const [password, setPassword] = useState('')  const [error, setError] = useState<Error | null>(null)  const history = useHistory()  async function handleSubmit(event: React.FormEvent<HTMLFormElement>) {    event.preventDefault()    try {      await login(username, password)      history.push('/')    } catch (error: unknown) {      setError(error as Error)    }  }  return (    <form onSubmit={handleSubmit}>      {/* ... */}    </form>  );}
```

note

When using the exposed `login()` function, make sure to implement your redirect on success login logic (e.g. redirecting to home).

#### `signup()`[​](#signup "Direct link to signup")

An action for signing up the user. This action does not log in the user, you still need to call `login()`.

It takes one argument:

-   `userFields: object` required
    
    It has the following fields:
    
    -   `username: string` required
        
    -   `password: string` required
        
    
    info
    
    By default, Wasp will only save the `username` and `password` fields. If you want to add extra fields to your signup process, read about [defining extra signup fields](/docs/auth/overview#customizing-the-signup-process).
    

You can use it like this:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import { signup, login } from 'wasp/client/auth'import { useState } from 'react'import { useHistory } from 'react-router-dom'import { Link } from 'react-router-dom'export function Signup() {  const [username, setUsername] = useState('')  const [password, setPassword] = useState('')  const [error, setError] = useState(null)  const history = useHistory()  async function handleSubmit(event) {    event.preventDefault()    try {      await signup({        username,        password,      })      await login(username, password)      history.push("/")    } catch (error) {      setError(error)    }  }  return (    <form onSubmit={handleSubmit}>      {/* ... */}    </form>  );}
```

src/pages/auth.tsx

```
    import { signup, login } from 'wasp/client/auth'import { useState } from 'react'import { useHistory } from 'react-router-dom'import { Link } from 'react-router-dom'export function Signup() {  const [username, setUsername] = useState('')  const [password, setPassword] = useState('')  const [error, setError] = useState<Error | null>(null)  const history = useHistory()  async function handleSubmit(event: React.FormEvent<HTMLFormElement>) {    event.preventDefault()    try {      await signup({        username,        password,      })      await login(username, password)      history.push("/")    } catch (error: unknown) {      setError(error as Error)    }  }  return (    <form onSubmit={handleSubmit}>      {/* ... */}    </form>  );}
```

### 2\. Creating your custom sign-up action[​](#2-creating-your-custom-sign-up-action "Direct link to 2. Creating your custom sign-up action")

The code of your custom sign-up action can look like this:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...action customSignup {  fn: import { signup } from "@src/auth/signup.js",}
```

src/auth/signup.js

```
    import {  ensurePasswordIsPresent,  ensureValidPassword,  ensureValidUsername,  createProviderId,  sanitizeAndSerializeProviderData,  createUser,} from 'wasp/server/auth'export const signup = async (args, _context) => {  ensureValidUsername(args)  ensurePasswordIsPresent(args)  ensureValidPassword(args)  try {    const providerId = createProviderId('username', args.username)    const providerData = await sanitizeAndSerializeProviderData({      hashedPassword: args.password,    })    await createUser(      providerId,      providerData,      // Any additional data you want to store on the User entity      {},    )  } catch (e) {    return {      success: false,      message: e.message,    }  }  // Your custom code after sign-up.  // ...  return {    success: true,    message: 'User created successfully',  }}
```

main.wasp

```
    // ...action customSignup {  fn: import { signup } from "@src/auth/signup.js",}
```

src/auth/signup.ts

```
    import {  ensurePasswordIsPresent,  ensureValidPassword,  ensureValidUsername,  createProviderId,  sanitizeAndSerializeProviderData,  createUser,} from 'wasp/server/auth'import type { CustomSignup } from 'wasp/server/operations'type CustomSignupInput = {  username: string  password: string}type CustomSignupOutput = {  success: boolean  message: string}export const signup: CustomSignup<  CustomSignupInput,  CustomSignupOutput> = async (args, _context) => {  ensureValidUsername(args)  ensurePasswordIsPresent(args)  ensureValidPassword(args)  try {    const providerId = createProviderId('username', args.username)    const providerData = await sanitizeAndSerializeProviderData<'username'>({      hashedPassword: args.password,    })    await createUser(      providerId,      providerData,      // Any additional data you want to store on the User entity      {},    )  } catch (e) {    return {      success: false,      message: e.message,    }  }  // Your custom code after sign-up.  // ...  return {    success: true,    message: 'User created successfully',  }}
```

We suggest using the built-in field validators for your authentication flow. You can import them from `wasp/server/auth`. These are the same validators that Wasp uses internally for the default authentication flow.

#### Username[​](#username "Direct link to Username")

-   `ensureValidUsername(args)`
    
    Checks if the username is valid and throws an error if it's not. Read more about the validation rules [here](/docs/auth/overview#default-validations).
    

#### Password[​](#password "Direct link to Password")

-   `ensurePasswordIsPresent(args)`
    
    Checks if the password is present and throws an error if it's not.
    
-   `ensureValidPassword(args)`
    
    Checks if the password is valid and throws an error if it's not. Read more about the validation rules [here](/docs/auth/overview#default-validations).
    

## Using Auth[​](#using-auth "Direct link to Using Auth")

To read more about how to set up the logout button and how to get access to the logged-in user in our client and server code, read the [auth overview docs](/docs/auth/overview).

When you receive the `user` object [on the client or the server](/docs/auth/overview#accessing-the-logged-in-user), you'll be able to access the user's username like this:

```
    const usernameIdentity = user.identities.username// Username that the user used to sign up, e.g. "fluffyllama"usernameIdentity.id
```

Read more about accessing the user data in the [Accessing User Data](/docs/auth/entities#accessing-the-auth-fields) section of the docs.

## API Reference[​](#api-reference "Direct link to API Reference")

### `userEntity` fields[​](#userentity-fields "Direct link to userentity-fields")

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      usernameAndPassword: {},    },    onAuthFailedRedirectTo: "/login"  }}
```

schema.prisma

```
    model User {  id Int @id @default(autoincrement())}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      usernameAndPassword: {},    },    onAuthFailedRedirectTo: "/login"  }}
```

schema.prisma

```
    model User {  id Int @id @default(autoincrement())}
```

The user entity needs to have the following fields:

-   `id` required
    
    It can be of any type, but it needs to be marked with `@id`
    

You can add any other fields you want to the user entity. Make sure to also define them in the `userSignupFields` field if they need to be set during the sign-up process.

### Fields in the `usernameAndPassword` dict[​](#fields-in-the-usernameandpassword-dict "Direct link to fields-in-the-usernameandpassword-dict")

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      usernameAndPassword: {        userSignupFields: import { userSignupFields } from "@src/auth/email.js",      },    },    onAuthFailedRedirectTo: "/login"  }}// ...
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      usernameAndPassword: {        userSignupFields: import { userSignupFields } from "@src/auth/email.js",      },    },    onAuthFailedRedirectTo: "/login"  }}// ...
```

#### `userSignupFields: ExtImport`[​](#usersignupfields-extimport "Direct link to usersignupfields-extimport")

`userSignupFields` defines all the extra fields that need to be set on the `User` during the sign-up process. For example, if you have `address` and `phone` fields on your `User` entity, you can set them by defining the `userSignupFields` like this:

-   JavaScript
-   TypeScript

src/auth.js

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
```

src/auth.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
```

Read more about the \`userSignupFields\` function \[here\](./overview#1-defining-extra-fields).

---

# Email

Wasp supports e-mail authentication out of the box, along with email verification and "forgot your password?" flows. It provides you with the server-side implementation and email templates for all of these flows.

![Auth UI](/assets/images/all_screens-4474bfaf188590d975e11d5b4e04b0ae.gif)

Using multiple auth identities for a single user

Wasp currently doesn't support multiple auth identities for a single user. This means, for example, that a user can't have both an email-based auth identity and a Google-based auth identity. This is something we will add in the future with the introduction of the [account merging feature](https://github.com/wasp-lang/wasp/issues/954).

Account merging means that multiple auth identities can be merged into a single user account. For example, a user's email and Google identity can be merged into a single user account. Then the user can log in with either their email or Google account and they will be logged into the same account.

## Setting Up Email Authentication[​](#setting-up-email-authentication "Direct link to Setting Up Email Authentication")

We'll need to take the following steps to set up email authentication:

1.  Enable email authentication in the Wasp file
2.  Add the `User` entity
3.  Add the auth routes and pages
4.  Use Auth UI components in our pages
5.  Set up the email sender

Structure of the `main.wasp` file we will end up with:

main.wasp

```
    // Configuring e-mail authenticationapp myApp {  auth: { ... }}// Defining routes and pagesroute SignupRoute { ... }page SignupPage { ... }// ...
```

### 1\. Enable Email Authentication in `main.wasp`[​](#1-enable-email-authentication-in-mainwasp "Direct link to 1-enable-email-authentication-in-mainwasp")

Let's start with adding the following to our `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the user entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable email authentication      email: {        // 3. Specify the email from field        fromField: {          name: "My App Postman",          email: "hello@itsme.com"        },        // 4. Specify the email verification and password reset options (we'll talk about them later)        emailVerification: {          clientRoute: EmailVerificationRoute,        },        passwordReset: {          clientRoute: PasswordResetRoute,        },      },    },    onAuthFailedRedirectTo: "/login",    onAuthSucceededRedirectTo: "/"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the user entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable email authentication      email: {        // 3. Specify the email from field        fromField: {          name: "My App Postman",          email: "hello@itsme.com"        },        // 4. Specify the email verification and password reset options (we'll talk about them later)        emailVerification: {          clientRoute: EmailVerificationRoute,        },        passwordReset: {          clientRoute: PasswordResetRoute,        },      },    },    onAuthFailedRedirectTo: "/login",    onAuthSucceededRedirectTo: "/"  },}
```

Read more about the `email` auth method options [here](#fields-in-the-email-dict).

### 2\. Add the User Entity[​](#2-add-the-user-entity "Direct link to 2. Add the User Entity")

The `User` entity can be as simple as including only the `id` field:

-   JavaScript
-   TypeScript

schema.prisma

```
    // 5. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

schema.prisma

```
    // 5. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

You can read more about how the `User` is connected to the rest of the auth system and how you can access the user data in the [Accessing User Data](/docs/auth/entities) section of the docs.

### 3\. Add the Routes and Pages[​](#3-add-the-routes-and-pages "Direct link to 3. Add the Routes and Pages")

Next, we need to define the routes and pages for the authentication pages.

Add the following to the `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.jsx"}route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { Signup } from "@src/pages/auth.jsx"}route RequestPasswordResetRoute { path: "/request-password-reset", to: RequestPasswordResetPage }page RequestPasswordResetPage {  component: import { RequestPasswordReset } from "@src/pages/auth.jsx",}route PasswordResetRoute { path: "/password-reset", to: PasswordResetPage }page PasswordResetPage {  component: import { PasswordReset } from "@src/pages/auth.jsx",}route EmailVerificationRoute { path: "/email-verification", to: EmailVerificationPage }page EmailVerificationPage {  component: import { EmailVerification } from "@src/pages/auth.jsx",}
```

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.tsx"}route SignupRoute { path: "/signup", to: SignupPage }page SignupPage {  component: import { Signup } from "@src/pages/auth.tsx"}route RequestPasswordResetRoute { path: "/request-password-reset", to: RequestPasswordResetPage }page RequestPasswordResetPage {  component: import { RequestPasswordReset } from "@src/pages/auth.tsx",}route PasswordResetRoute { path: "/password-reset", to: PasswordResetPage }page PasswordResetPage {  component: import { PasswordReset } from "@src/pages/auth.tsx",}route EmailVerificationRoute { path: "/email-verification", to: EmailVerificationPage }page EmailVerificationPage {  component: import { EmailVerification } from "@src/pages/auth.tsx",}
```

We'll define the React components for these pages in the `src/pages/auth.tsx` file below.

### 4\. Create the Client Pages[​](#4-create-the-client-pages "Direct link to 4. Create the Client Pages")

info

We are using [Tailwind CSS](https://tailwindcss.com/) to style the pages. Read more about how to add it [here](/docs/project/css-frameworks).

Let's create a `auth.tsx` file in the `src/pages` folder and add the following to it:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import {  LoginForm,  SignupForm,  VerifyEmailForm,  ForgotPasswordForm,  ResetPasswordForm,} from 'wasp/client/auth'import { Link } from 'react-router-dom'export function Login() {  return (    <Layout>      <LoginForm />      <br />      <span className="text-sm font-medium text-gray-900">        Don't have an account yet? <Link to="/signup">go to signup</Link>.      </span>      <br />      <span className="text-sm font-medium text-gray-900">        Forgot your password? <Link to="/request-password-reset">reset it</Link>        .      </span>    </Layout>  );}export function Signup() {  return (    <Layout>      <SignupForm />      <br />      <span className="text-sm font-medium text-gray-900">        I already have an account (<Link to="/login">go to login</Link>).      </span>    </Layout>  );}export function EmailVerification() {  return (    <Layout>      <VerifyEmailForm />      <br />      <span className="text-sm font-medium text-gray-900">        If everything is okay, <Link to="/login">go to login</Link>      </span>    </Layout>  );}export function RequestPasswordReset() {  return (    <Layout>      <ForgotPasswordForm />    </Layout>  );}export function PasswordReset() {  return (    <Layout>      <ResetPasswordForm />      <br />      <span className="text-sm font-medium text-gray-900">        If everything is okay, <Link to="/login">go to login</Link>      </span>    </Layout>  );}// A layout component to center the contentexport function Layout({ children }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  );}
```

src/pages/auth.tsx

```
    import {  LoginForm,  SignupForm,  VerifyEmailForm,  ForgotPasswordForm,  ResetPasswordForm,} from 'wasp/client/auth'import { Link } from 'react-router-dom'export function Login() {  return (    <Layout>      <LoginForm />      <br />      <span className="text-sm font-medium text-gray-900">        Don't have an account yet? <Link to="/signup">go to signup</Link>.      </span>      <br />      <span className="text-sm font-medium text-gray-900">        Forgot your password? <Link to="/request-password-reset">reset it</Link>        .      </span>    </Layout>  );}export function Signup() {  return (    <Layout>      <SignupForm />      <br />      <span className="text-sm font-medium text-gray-900">        I already have an account (<Link to="/login">go to login</Link>).      </span>    </Layout>  );}export function EmailVerification() {  return (    <Layout>      <VerifyEmailForm />      <br />      <span className="text-sm font-medium text-gray-900">        If everything is okay, <Link to="/login">go to login</Link>      </span>    </Layout>  );}export function RequestPasswordReset() {  return (    <Layout>      <ForgotPasswordForm />    </Layout>  );}export function PasswordReset() {  return (    <Layout>      <ResetPasswordForm />      <br />      <span className="text-sm font-medium text-gray-900">        If everything is okay, <Link to="/login">go to login</Link>      </span>    </Layout>  );}// A layout component to center the contentexport function Layout({ children }: { children: React.ReactNode }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  );}
```

We imported the generated Auth UI components and used them in our pages. Read more about the Auth UI components [here](/docs/auth/ui).

### 5\. Set up an Email Sender[​](#5-set-up-an-email-sender "Direct link to 5. Set up an Email Sender")

To support e-mail verification and password reset flows, we need an e-mail sender. Luckily, Wasp supports several email providers out of the box.

We'll use the `Dummy` provider to speed up the setup. It just logs the emails to the console instead of sending them. You can use any of the [supported email providers](/docs/advanced/email#providers).

To set up the `Dummy` provider to send emails, add the following to the `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  // ...  // 7. Set up the email sender  emailSender: {    provider: Dummy,  }}
```

main.wasp

```
    app myApp {  // ...  // 7. Set up the email sender  emailSender: {    provider: Dummy,  }}
```

### Conclusion[​](#conclusion "Direct link to Conclusion")

That's it! We have set up email authentication in our app. 🎉

Running `wasp db migrate-dev` and then `wasp start` should give you a working app with email authentication. If you want to put some of the pages behind authentication, read the [auth overview](/docs/auth/overview).

## Login and Signup Flows[​](#login-and-signup-flows "Direct link to Login and Signup Flows")

### Login[​](#login "Direct link to Login")

![Auth UI](/assets/images/login-1dad984cebbef0e4bd2bc0c008d2d2ff.png)

### Signup[​](#signup "Direct link to Signup")

![Auth UI](/assets/images/signup-4362e29083e15457eb5a16990f57819a.png)

Some of the behavior you get out of the box:

1.  Rate limiting
    
    We are limiting the rate of sign-up requests to **1 request per minute** per email address. This is done to prevent spamming.
    
2.  Preventing user email leaks
    
    If somebody tries to signup with an email that already exists and it's verified, we *pretend* that the account was created instead of saying it's an existing account. This is done to prevent leaking the user's email address.
    
3.  Allowing registration for unverified emails
    
    If a user tries to register with an existing but **unverified** email, we'll allow them to do that. This is done to prevent bad actors from locking out other users from registering with their email address.
    
4.  Password validation
    
    Read more about the default password validation rules and how to override them in [auth overview docs](/docs/auth/overview).
    

## Email Verification Flow[​](#email-verification-flow "Direct link to Email Verification Flow")

Automatic email verification in development

In development mode, you can skip the email verification step by setting the `SKIP_EMAIL_VERIFICATION_IN_DEV` environment variable to `true` in your `.env.server` file:

.env.server

```
    SKIP_EMAIL_VERIFICATION_IN_DEV=true
```

This is useful when you are developing your app and don't want to go through the email verification flow every time you sign up. It can be also useful when you are writing automated tests for your app.

By default, Wasp requires the e-mail to be verified before allowing the user to log in. This is done by sending a verification email to the user's email address and requiring the user to click on a link in the email to verify their email address.

Our setup looks like this:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...emailVerification: {    clientRoute: EmailVerificationRoute,}
```

main.wasp

```
    // ...emailVerification: {    clientRoute: EmailVerificationRoute,}
```

When the user receives an e-mail, they receive a link that goes to the client route specified in the `clientRoute` field. In our case, this is the `EmailVerificationRoute` route we defined in the `main.wasp` file.

The content of the e-mail can be customized, read more about it [here](#emailverification-emailverificationconfig-).

### Email Verification Page[​](#email-verification-page "Direct link to Email Verification Page")

We defined our email verification page in the `auth.tsx` file.

![Auth UI](/assets/images/email_verification-b1919c242edc678175636485f6fc2264.png)

## Password Reset Flow[​](#password-reset-flow "Direct link to Password Reset Flow")

Users can request a password and then they'll receive an e-mail with a link to reset their password.

Some of the behavior you get out of the box:

1.  Rate limiting
    
    We are limiting the rate of sign-up requests to **1 request per minute** per email address. This is done to prevent spamming.
    
2.  Preventing user email leaks
    
    If somebody requests a password reset with an unknown email address, we'll give back the same response as if the user requested a password reset successfully. This is done to prevent leaking information.
    

Our setup in `main.wasp` looks like this:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...passwordReset: {    clientRoute: PasswordResetRoute,}
```

main.wasp

```
    // ...passwordReset: {    clientRoute: PasswordResetRoute,}
```

### Request Password Reset Page[​](#request-password-reset-page "Direct link to Request Password Reset Page")

Users request their password to be reset by going to the `/request-password-reset` route. We defined our request password reset page in the `auth.tsx` file.

![Request password reset page](/assets/images/forgot_password_after-db85e082ff373ebb95f3bdf9e64c1918.png)

### Password Reset Page[​](#password-reset-page "Direct link to Password Reset Page")

When the user receives an e-mail, they receive a link that goes to the client route specified in the `clientRoute` field. In our case, this is the `PasswordResetRoute` route we defined in the `main.wasp` file.

![Request password reset page](/assets/images/reset_password_after-3eecfee765304fd41b6385519cc5e522.png)

Users can enter their new password there.

The content of the e-mail can be customized, read more about it [here](#passwordreset-passwordresetconfig-).

## Creating a Custom Sign-up Action[​](#creating-a-custom-sign-up-action "Direct link to Creating a Custom Sign-up Action")

Creating a custom sign-up action

We don't recommend creating a custom sign-up action unless you have a good reason to do so. It is a complex process and you can easily make a mistake that will compromise the security of your app.

The code of your custom sign-up action can look like this:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...action customSignup {  fn: import { signup } from "@src/auth/signup.js",}
```

src/auth/signup.js

```
    import {  ensurePasswordIsPresent,  ensureValidPassword,  ensureValidEmail,  createProviderId,  sanitizeAndSerializeProviderData,  deserializeAndSanitizeProviderData,  findAuthIdentity,  createUser,  createEmailVerificationLink,  sendEmailVerificationEmail,} from 'wasp/server/auth'export const signup = async (args, _context) => {  ensureValidEmail(args)  ensurePasswordIsPresent(args)  ensureValidPassword(args)  try {    const providerId = createProviderId('email', args.email)    const existingAuthIdentity = await findAuthIdentity(providerId)    if (existingAuthIdentity) {      const providerData = deserializeAndSanitizeProviderData(existingAuthIdentity.providerData)      // Your custom code here    } else {      // sanitizeAndSerializeProviderData will hash the user's password      const newUserProviderData = await sanitizeAndSerializeProviderData({          hashedPassword: args.password,          isEmailVerified: false,          emailVerificationSentAt: null,          passwordResetSentAt: null,      })      await createUser(        providerId,        providerData,        // Any additional data you want to store on the User entity        {},      )      // Verification link links to a client route e.g. /email-verification      const verificationLink = await createEmailVerificationLink(args.email, '/email-verification');      try {          await sendEmailVerificationEmail(              args.email,              {                  from: {                    name: "My App Postman",                    email: "hello@itsme.com",                  },                  to: args.email,                  subject: "Verify your email",                  text: `Click the link below to verify your email: ${verificationLink}`,                  html: `                      <p>Click the link below to verify your email</p>                      <a href="${verificationLink}">Verify email</a>                  `,              }          );      } catch (e: unknown) {          console.error("Failed to send email verification email:", e);          throw new HttpError(500, "Failed to send email verification email.");      }     }  } catch (e) {    return {      success: false,      message: e.message,    }  }  // Your custom code after sign-up.  // ...  return {    success: true,    message: 'User created successfully',  }}
```

main.wasp

```
    // ...action customSignup {  fn: import { signup } from "@src/auth/signup.js",}
```

src/auth/signup.ts

```
    import {  ensurePasswordIsPresent,  ensureValidPassword,  ensureValidEmail,  createProviderId,  sanitizeAndSerializeProviderData,  deserializeAndSanitizeProviderData,  findAuthIdentity,  createUser,  createEmailVerificationLink,  sendEmailVerificationEmail,} from 'wasp/server/auth'import type { CustomSignup } from 'wasp/server/operations'type CustomSignupInput = {  email: string  password: string}type CustomSignupOutput = {  success: boolean  message: string}export const signup: CustomSignup<CustomSignupInput, CustomSignupOutput> = async (args, _context) => {  ensureValidEmail(args)  ensurePasswordIsPresent(args)  ensureValidPassword(args)  try {    const providerId = createProviderId('email', args.email)    const existingAuthIdentity = await findAuthIdentity(providerId)    if (existingAuthIdentity) {      const providerData = deserializeAndSanitizeProviderData<'email'>(existingAuthIdentity.providerData)      // Your custom code here    } else {      // sanitizeAndSerializeProviderData will hash the user's password      const newUserProviderData = await sanitizeAndSerializeProviderData<'email'>({          hashedPassword: args.password,          isEmailVerified: false,          emailVerificationSentAt: null,          passwordResetSentAt: null,      })      await createUser(        providerId,        providerData,        // Any additional data you want to store on the User entity        {},      )      // Verification link links to a client route e.g. /email-verification      const verificationLink = await createEmailVerificationLink(args.email, '/email-verification');      try {          await sendEmailVerificationEmail(              args.email,              {                  from: {                    name: "My App Postman",                    email: "hello@itsme.com",                  },                  to: args.email,                  subject: "Verify your email",                  text: `Click the link below to verify your email: ${verificationLink}`,                  html: `                      <p>Click the link below to verify your email</p>                      <a href="${verificationLink}">Verify email</a>                  `,              }          );      } catch (e: unknown) {          console.error("Failed to send email verification email:", e);          throw new HttpError(500, "Failed to send email verification email.");      }     }  } catch (e) {    return {      success: false,      message: e.message,    }  }  // Your custom code after sign-up.  // ...  return {    success: true,    message: 'User created successfully',  }}
```

We suggest using the built-in field validators for your authentication flow. You can import them from `wasp/server/auth`. These are the same validators that Wasp uses internally for the default authentication flow.

#### Email[​](#email "Direct link to Email")

-   `ensureValidEmail(args)`
    
    Checks if the email is valid and throws an error if it's not. Read more about the validation rules [here](/docs/auth/overview#default-validations).
    

#### Password[​](#password "Direct link to Password")

-   `ensurePasswordIsPresent(args)`
    
    Checks if the password is present and throws an error if it's not.
    
-   `ensureValidPassword(args)`
    
    Checks if the password is valid and throws an error if it's not. Read more about the validation rules [here](/docs/auth/overview#default-validations).
    

## Using Auth[​](#using-auth "Direct link to Using Auth")

To read more about how to set up the logout button and how to get access to the logged-in user in our client and server code, read the [auth overview docs](/docs/auth/overview).

When you receive the `user` object [on the client or the server](/docs/auth/overview#accessing-the-logged-in-user), you'll be able to access the user's email and other information like this:

```
    const emailIdentity = user.identities.email// Email address the the user used to sign up, e.g. "fluffyllama@app.com".emailIdentity.id// `true` if the user has verified their email address.emailIdentity.isEmailVerified// Datetime when the email verification email was sent.emailIdentity.emailVerificationSentAt// Datetime when the last password reset email was sent.emailIdentity.passwordResetSentAt
```

Read more about accessing the user data in the [Accessing User Data](/docs/auth/entities#accessing-the-auth-fields) section of the docs.

## API Reference[​](#api-reference "Direct link to API Reference")

Let's go over the options we can specify when using email authentication.

### `userEntity` fields[​](#userentity-fields "Direct link to userentity-fields")

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  title: "My app",  // ...  auth: {    userEntity: User,    methods: {      email: {        // We'll explain these options below      },    },    onAuthFailedRedirectTo: "/someRoute"  },  // ...}
```

schema.prisma

```
    model User {  id Int @id @default(autoincrement())}
```

main.wasp

```
    app myApp {  title: "My app",  // ...  auth: {    userEntity: User,    methods: {      email: {        // We'll explain these options below      },    },    onAuthFailedRedirectTo: "/someRoute"  },  // ...}
```

schema.prisma

```
    model User {  id Int @id @default(autoincrement())}
```

The user entity needs to have the following fields:

-   `id` required
    
    It can be of any type, but it needs to be marked with `@id`
    

You can add any other fields you want to the user entity. Make sure to also define them in the `userSignupFields` field if they need to be set during the sign-up process.

### Fields in the `email` dict[​](#fields-in-the-email-dict "Direct link to fields-in-the-email-dict")

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  title: "My app",  // ...  auth: {    userEntity: User,    methods: {      email: {        userSignupFields: import { userSignupFields } from "@src/auth.js",        fromField: {          name: "My App",          email: "hello@itsme.com"        },        emailVerification: {          clientRoute: EmailVerificationRoute,          getEmailContentFn: import { getVerificationEmailContent } from "@src/auth/email.js",        },        passwordReset: {          clientRoute: PasswordResetRoute,          getEmailContentFn: import { getPasswordResetEmailContent } from "@src/auth/email.js",        },      },    },    onAuthFailedRedirectTo: "/someRoute"  },  // ...}
```

main.wasp

```
    app myApp {  title: "My app",  // ...  auth: {    userEntity: User,    methods: {      email: {        userSignupFields: import { userSignupFields } from "@src/auth.js",        fromField: {          name: "My App",          email: "hello@itsme.com"        },        emailVerification: {          clientRoute: EmailVerificationRoute,          getEmailContentFn: import { getVerificationEmailContent } from "@src/auth/email.js",        },        passwordReset: {          clientRoute: PasswordResetRoute,          getEmailContentFn: import { getPasswordResetEmailContent } from "@src/auth/email.js",        },      },    },    onAuthFailedRedirectTo: "/someRoute"  },  // ...}
```

#### `userSignupFields: ExtImport`[​](#usersignupfields-extimport "Direct link to usersignupfields-extimport")

`userSignupFields` defines all the extra fields that need to be set on the `User` during the sign-up process. For example, if you have `address` and `phone` fields on your `User` entity, you can set them by defining the `userSignupFields` like this:

-   JavaScript
-   TypeScript

src/auth.js

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
```

src/auth.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
```

Read more about the `userSignupFields` function [here](/docs/auth/overview#1-defining-extra-fields).

#### `fromField: EmailFromField` required[​](#fromfield-emailfromfield- "Direct link to fromfield-emailfromfield-")

`fromField` is a dict that specifies the name and e-mail address of the sender of the e-mails sent by your app.

It has the following fields:

-   `name`: name of the sender
-   `email`: e-mail address of the sender required

#### `emailVerification: EmailVerificationConfig` required[​](#emailverification-emailverificationconfig- "Direct link to emailverification-emailverificationconfig-")

`emailVerification` is a dict that specifies the details of the e-mail verification process.

It has the following fields:

-   `clientRoute: Route`: a route that is used for the user to verify their e-mail address. required
    
    Client route should handle the process of taking a token from the URL and sending it to the server to verify the e-mail address. You can use our `verifyEmail` action for that.
    
    -   JavaScript
    -   TypeScript
    
    src/pages/EmailVerificationPage.jsx
    
    ```
        import { verifyEmail } from 'wasp/client/auth'...await verifyEmail({ token });
    ```
    
    src/pages/EmailVerificationPage.tsx
    
    ```
        import { verifyEmail } from 'wasp/client/auth'...await verifyEmail({ token });
    ```
    
    note
    
    We used Auth UI above to avoid doing this work of sending the token to the server manually.
    
-   `getEmailContentFn: ExtImport`: a function that returns the content of the e-mail that is sent to the user.
    
    Defining `getEmailContentFn` can be done by defining a file in the `src` directory.
    
    -   JavaScript
    -   TypeScript
    
    src/email.js
    
    ```
        export const getVerificationEmailContent = ({ verificationLink }) => ({  subject: 'Verify your email',  text: `Click the link below to verify your email: ${verificationLink}`,  html: `        <p>Click the link below to verify your email</p>        <a href="${verificationLink}">Verify email</a>    `,})
    ```
    
    src/email.ts
    
    ```
        import { GetVerificationEmailContentFn } from 'wasp/server/auth'export const getVerificationEmailContent: GetVerificationEmailContentFn = ({  verificationLink,}) => ({  subject: 'Verify your email',  text: `Click the link below to verify your email: ${verificationLink}`,  html: `        <p>Click the link below to verify your email</p>        <a href="${verificationLink}">Verify email</a>    `,})
    ```
    
    This is the default content of the e-mail, you can customize it to your liking.

#### `passwordReset: PasswordResetConfig` required[​](#passwordreset-passwordresetconfig- "Direct link to passwordreset-passwordresetconfig-")

`passwordReset` is a dict that specifies the password reset process.

It has the following fields:

-   `clientRoute: Route`: a route that is used for the user to reset their password. required
    
    Client route should handle the process of taking a token from the URL and a new password from the user and sending it to the server. You can use our `requestPasswordReset` and `resetPassword` actions to do that.
    
    -   JavaScript
    -   TypeScript
    
    src/pages/ForgotPasswordPage.jsx
    
    ```
        import { requestPasswordReset } from 'wasp/client/auth'...await requestPasswordReset({ email });
    ```
    
    src/pages/PasswordResetPage.jsx
    
    ```
        import { resetPassword } from 'wasp/client/auth'...await resetPassword({ password, token })
    ```
    
    src/pages/ForgotPasswordPage.tsx
    
    ```
        import { requestPasswordReset } from 'wasp/client/auth'...await requestPasswordReset({ email });
    ```
    
    src/pages/PasswordResetPage.tsx
    
    ```
        import { resetPassword } from 'wasp/client/auth'...await resetPassword({ password, token })
    ```
    
    note
    
    We used Auth UI above to avoid doing this work of sending the password request and the new password to the server manually.
    
-   `getEmailContentFn: ExtImport`: a function that returns the content of the e-mail that is sent to the user.
    
    Defining `getEmailContentFn` is done by defining a function that looks like this:
    
    -   JavaScript
    -   TypeScript
    
    src/email.js
    
    ```
        export const getPasswordResetEmailContent = ({ passwordResetLink }) => ({  subject: 'Password reset',  text: `Click the link below to reset your password: ${passwordResetLink}`,  html: `        <p>Click the link below to reset your password</p>        <a href="${passwordResetLink}">Reset password</a>    `,})
    ```
    
    src/email.ts
    
    ```
        import { GetPasswordResetEmailContentFn } from 'wasp/server/auth'export const getPasswordResetEmailContent: GetPasswordResetEmailContentFn = ({  passwordResetLink,}) => ({  subject: 'Password reset',  text: `Click the link below to reset your password: ${passwordResetLink}`,  html: `        <p>Click the link below to reset your password</p>        <a href="${passwordResetLink}">Reset password</a>    `,})
    ```
    
    This is the default content of the e-mail, you can customize it to your liking.

---

# Overview

Social login options (e.g., *Log in with Google*) are a great (maybe even the best) solution for handling user accounts. A famous old developer joke tells us *"The best auth system is the one you never have to make."*

Wasp wants to make adding social login options to your app as painless as possible.

Using different social providers gives users a chance to sign into your app via their existing accounts on other platforms (Google, GitHub, etc.).

This page goes through the common behaviors between all supported social login providers and shows you how to customize them. It also gives an overview of Wasp's UI helpers - the quickest possible way to get started with social auth.

## Available Providers[​](#available-providers "Direct link to Available Providers")

Wasp currently supports the following social login providers:

[

### Google »

Users sign in with their Google account.

](/docs/auth/social-auth/google)[

### Github »

Users sign in with their Github account.

](/docs/auth/social-auth/github)[

### Keycloak »

Users sign in with their Keycloak account.

](/docs/auth/social-auth/keycloak)[

### Discord »

Users sign in with their Discord account.

](/docs/auth/social-auth/discord)

Click on each provider for more details.

## User Entity[​](#user-entity "Direct link to User Entity")

Wasp requires you to declare a `userEntity` for all `auth` methods (social or otherwise). This field tells Wasp which Entity represents the user.

Here's what the full setup looks like:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id Int @id @default(autoincrement())}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id Int @id @default(autoincrement())}
```

## Default Behavior[​](#default-behavior "Direct link to Default Behavior")

When a user **signs in for the first time**, Wasp creates a new user account and links it to the chosen auth provider account for future logins.

## Overrides[​](#overrides "Direct link to Overrides")

By default, Wasp doesn't store any information it receives from the social login provider. It only stores the user's ID specific to the provider.

If you wish to store more information about the user, you can override the default behavior. You can do this by defining the `userSignupFields` and `configFn` fields in `main.wasp` for each provider.

You can create custom signup setups, such as allowing users to define a custom username after they sign up with a social provider.

### Example: Allowing User to Set Their Username[​](#example-allowing-user-to-set-their-username "Direct link to Example: Allowing User to Set Their Username")

If you want to modify the signup flow (e.g., let users choose their own usernames), you will need to go through three steps:

1.  The first step is adding a `isSignupComplete` property to your `User` Entity. This field will signal whether the user has completed the signup process.
2.  The second step is overriding the default signup behavior.
3.  The third step is implementing the rest of your signup flow and redirecting users where appropriate.

Let's go through both steps in more detail.

#### 1\. Adding the `isSignupComplete` Field to the `User` Entity[​](#1-adding-the-issignupcomplete-field-to-the-user-entity "Direct link to 1-adding-the-issignupcomplete-field-to-the-user-entity")

-   JavaScript
-   TypeScript

schema.prisma

```
    model User {  id               Int     @id @default(autoincrement())  username         String? @unique  isSignupComplete Boolean @default(false)}
```

schema.prisma

```
    model User {  id               Int     @id @default(autoincrement())  username         String? @unique  isSignupComplete Boolean @default(false)}
```

#### 2\. Overriding the Default Behavior[​](#2-overriding-the-default-behavior "Direct link to 2. Overriding the Default Behavior")

Declare an import under `app.auth.methods.google.userSignupFields` (the example assumes you're using Google):

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {        userSignupFields: import { userSignupFields } from "@src/auth/google.js"      }    },    onAuthFailedRedirectTo: "/login"  },}// ...
```

And implement the imported function.

src/auth/google.js

```
    export const userSignupFields = {  isSignupComplete: () => false,}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {        userSignupFields: import { userSignupFields } from "@src/auth/google.js"      }    },    onAuthFailedRedirectTo: "/login"  },}// ...
```

And implement the imported function:

src/auth/google.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  isSignupComplete: () => false,})
```

Wasp automatically generates the `defineUserSignupFields` function to help you correctly type your `userSignupFields` object.

#### 3\. Showing the Correct State on the Client[​](#3-showing-the-correct-state-on-the-client "Direct link to 3. Showing the Correct State on the Client")

You can query the user's `isSignupComplete` flag on the client with the [`useAuth()`](/docs/auth/overview) hook. Depending on the flag's value, you can redirect users to the appropriate signup step.

For example:

1.  When the user lands on the homepage, check the value of `user.isSignupComplete`.
2.  If it's `false`, it means the user has started the signup process but hasn't yet chosen their username. Therefore, you can redirect them to `EditUserDetailsPage` where they can edit the `username` property.

-   JavaScript
-   TypeScript

src/HomePage.jsx

```
    import { useAuth } from 'wasp/client/auth'import { Redirect } from 'react-router-dom'export function HomePage() {  const { data: user } = useAuth()  if (user.isSignupComplete === false) {    return <Redirect to="/edit-user-details" />  }  // ...}
```

src/HomePage.tsx

```
    import { useAuth } from 'wasp/client/auth'import { Redirect } from 'react-router-dom'export function HomePage() {  const { data: user } = useAuth()  if (user.isSignupComplete === false) {    return <Redirect to="/edit-user-details" />  }  // ...}
```

The same general principle applies to more complex signup procedures, just change the boolean `isSignupComplete` property to a property like `currentSignupStep` that can hold more values.

### Using the User's Provider Account Details[​](#using-the-users-provider-account-details "Direct link to Using the User's Provider Account Details")

Account details are provider-specific. Each provider has their own rules for defining the `userSignupFields` and `configFn` fields:

[

### Google »

Users sign in with their Google account.

](/docs/auth/social-auth/google#overrides)[

### Github »

Users sign in with their Github account.

](/docs/auth/social-auth/github#overrides)[

### Keycloak »

Users sign in with their Keycloak account.

](/docs/auth/social-auth/keycloak#overrides)[

### Discord »

Users sign in with their Discord account.

](/docs/auth/social-auth/discord#overrides)

Click on each provider for more details.

## UI Helpers[​](#ui-helpers "Direct link to UI Helpers")

Use Auth UI

[Auth UI](/docs/auth/ui) is a common name for all high-level auth forms that come with Wasp.

These include fully functional auto-generated login and signup forms with working social login buttons. If you're looking for the fastest way to get your auth up and running, that's where you should look.

The UI helpers described below are lower-level and are useful for creating your custom forms.

Wasp provides sign-in buttons and URLs for each of the supported social login providers.

-   JavaScript
-   TypeScript

src/LoginPage.jsx

```
    import {  GoogleSignInButton,  googleSignInUrl,  GitHubSignInButton,  gitHubSignInUrl,} from 'wasp/client/auth'export const LoginPage = () => {  return (    <>      <GoogleSignInButton />      <GitHubSignInButton />      {/* or */}      <a href={googleSignInUrl}>Sign in with Google</a>      <a href={gitHubSignInUrl}>Sign in with GitHub</a>    </>  )}
```

src/LoginPage.tsx

```
    import {  GoogleSignInButton,  googleSignInUrl,  GitHubSignInButton,  gitHubSignInUrl,} from 'wasp/client/auth'export const LoginPage = () => {  return (    <>      <GoogleSignInButton />      <GitHubSignInButton />      {/* or */}      <a href={googleSignInUrl}>Sign in with Google</a>      <a href={gitHubSignInUrl}>Sign in with GitHub</a>    </>  )}
```

If you need even more customization, you can create your custom components using `signInUrl`s.

## API Reference[​](#api-reference "Direct link to API Reference")

### Fields in the `app.auth` Dictionary and Overrides[​](#fields-in-the-appauth-dictionary-and-overrides "Direct link to fields-in-the-appauth-dictionary-and-overrides")

For more information on:

-   Allowed fields in `app.auth`
-   `userSignupFields` and `configFn` functions

Check the provider-specific API References:

[

### Google »

Users sign in with their Google account.

](/docs/auth/social-auth/google#api-reference)[

### Github »

Users sign in with their Github account.

](/docs/auth/social-auth/github#api-reference)[

### Keycloak »

Users sign in with their Keycloak account.

](/docs/auth/social-auth/keycloak#api-reference)[

### Discord »

Users sign in with their Discord account.

](/docs/auth/social-auth/discord#api-reference)

Click on each provider for more details.

---

# GitHub

Wasp supports Github Authentication out of the box. GitHub is a great external auth choice when you're building apps for developers, as most of them already have a GitHub account.

Letting your users log in using their GitHub accounts turns the signup process into a breeze.

Let's walk through enabling Github Authentication, explain some of the default settings, and show how to override them.

## Setting up Github Auth[​](#setting-up-github-auth "Direct link to Setting up Github Auth")

Enabling GitHub Authentication comes down to a series of steps:

1.  Enabling GitHub authentication in the Wasp file.
2.  Adding the `User` entity.
3.  Creating a GitHub OAuth app.
4.  Adding the necessary Routes and Pages
5.  Using Auth UI components in our Pages.

Here's a skeleton of how our `main.wasp` should look like after we're done:

main.wasp

```
    // Configuring the social authenticationapp myApp {  auth: { ... }}// Defining routes and pagesroute LoginRoute { ... }page LoginPage { ... }
```

### 1\. Adding Github Auth to Your Wasp File[​](#1-adding-github-auth-to-your-wasp-file "Direct link to 1. Adding Github Auth to Your Wasp File")

Let's start by properly configuring the Auth object:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the User entity  (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Github Auth      gitHub: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the User entity  (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Github Auth      gitHub: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

### 2\. Add the User Entity[​](#2-add-the-user-entity "Direct link to 2. Add the User Entity")

Let's now define the `app.auth.userEntity` entity in the `schema.prisma` file:

-   JavaScript
-   TypeScript

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

### 3\. Creating a GitHub OAuth App[​](#3-creating-a-github-oauth-app "Direct link to 3. Creating a GitHub OAuth App")

To use GitHub as an authentication method, you'll first need to create a GitHub OAuth App and provide Wasp with your client key and secret. Here's how you do it:

1.  Log into your GitHub account and navigate to: [https://github.com/settings/developers](https://github.com/settings/developers).
2.  Select **New OAuth App**.
3.  Supply required information.

![GitHub Applications Screenshot](/img/integrations-github-1.png)

-   For **Authorization callback URL**:
    -   For development, put: `http://localhost:3001/auth/github/callback`.
    -   Once you know on which URL your API server will be deployed, you can create a new app with that URL instead e.g. `https://your-server-url.com/auth/github/callback`.

4.  Hit **Register application**.
5.  Hit **Generate a new client secret** on the next page.
6.  Copy your Client ID and Client secret as you'll need them in the next step.

### 4\. Adding Environment Variables[​](#4-adding-environment-variables "Direct link to 4. Adding Environment Variables")

Add these environment variables to the `.env.server` file at the root of your project (take their values from the previous step):

.env.server

```
    GITHUB_CLIENT_ID=your-github-client-idGITHUB_CLIENT_SECRET=your-github-client-secret
```

### 5\. Adding the Necessary Routes and Pages[​](#5-adding-the-necessary-routes-and-pages "Direct link to 5. Adding the Necessary Routes and Pages")

Let's define the necessary authentication Routes and Pages.

Add the following code to your `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.jsx"}
```

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.tsx"}
```

We'll define the React components for these pages in the `src/pages/auth.tsx` file below.

### 6\. Creating the Client Pages[​](#6-creating-the-client-pages "Direct link to 6. Creating the Client Pages")

info

We are using [Tailwind CSS](https://tailwindcss.com/) to style the pages. Read more about how to add it [here](/docs/project/css-frameworks).

Let's create a `auth.tsx` file in the `src/pages` folder and add the following to it:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

src/pages/auth.tsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }: { children: React.ReactNode }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

We imported the generated Auth UI components and used them in our pages. Read more about the Auth UI components [here](/docs/auth/ui).

### Conclusion[​](#conclusion "Direct link to Conclusion")

Yay, we've successfully set up Github Auth! 🎉

![Github Auth](/assets/images/github-e4d12d3e83fdbfbf93ce1fa831645c0e.png)

Running `wasp db migrate-dev` and `wasp start` should now give you a working app with authentication. To see how to protect specific pages (i.e., hide them from non-authenticated users), read the docs on [using auth](/docs/auth/overview).

## Default Behaviour[​](#default-behaviour "Direct link to Default Behaviour")

Add `gitHub: {}` to the `auth.methods` dictionary to use it with default settings.

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      gitHub: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      gitHub: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

When a user **signs in for the first time**, Wasp creates a new user account and links it to the chosen auth provider account for future logins.

## Overrides[​](#overrides "Direct link to Overrides")

By default, Wasp doesn't store any information it receives from the social login provider. It only stores the user's ID specific to the provider.

There are two mechanisms used for overriding the default behavior:

-   `userSignupFields`
-   `configFn`

Let's explore them in more detail.

### Data Received From GitHub[​](#data-received-from-github "Direct link to Data Received From GitHub")

We are using GitHub's API and its `/user` and `/user/emails` endpoints to get the user data.

We combine the data from the two endpoints

You'll find the emails in the `emails` property in the object that you receive in `userSignupFields`.

This is because we combine the data from the `/user` and `/user/emails` endpoints **if the `user` or `user:email` scope is requested.**

The data we receive from GitHub on the `/user` endpoint looks something this:

```
    {  "login": "octocat",  "id": 1,  "name": "monalisa octocat",  "avatar_url": "https://github.com/images/error/octocat_happy.gif",  "gravatar_id": "",  // ...}
```

And the data from the `/user/emails` endpoint looks something like this:

```
    [  {    "email": "octocat@github.com",    "verified": true,    "primary": true,    "visibility": "public"  }]
```

The fields you receive will depend on the scopes you requested. By default we don't specify any scopes. If you want to get the emails, you need to specify the `user` or `user:email` scope in the `configFn` function.

For an up to date info about the data received from GitHub, please refer to the [GitHub API documentation](https://docs.github.com/en/rest/users/users?apiVersion=2022-11-28#get-the-authenticated-user).

### Using the Data Received From GitHub[​](#using-the-data-received-from-github "Direct link to Using the Data Received From GitHub")

When a user logs in using a social login provider, the backend receives some data about the user. Wasp lets you access this data inside the `userSignupFields` getters.

For example, the User entity can include a `displayName` field which you can set based on the details received from the provider.

Wasp also lets you customize the configuration of the providers' settings using the `configFn` function.

Let's use this example to show both fields in action:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      gitHub: {        configFn: import { getConfig } from "@src/auth/github.js",        userSignupFields: import { userSignupFields } from "@src/auth/github.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/github.js

```
    export const userSignupFields = {  username: () => "hardcoded-username",  displayName: (data) => data.profile.name,};export function getConfig() {  return {    scopes: ['user'],  };}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      gitHub: {        configFn: import { getConfig } from "@src/auth/github.js",        userSignupFields: import { userSignupFields } from "@src/auth/github.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/github.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  username: () => "hardcoded-username",  displayName: (data: any) => data.profile.name,})export function getConfig() {  return {    scopes: ['user'],  }}
```

Wasp automatically generates the `defineUserSignupFields` function to help you correctly type your `userSignupFields` object.

## Using Auth[​](#using-auth "Direct link to Using Auth")

To read more about how to set up the logout button and get access to the logged-in user in both client and server code, read the docs on [using auth](/docs/auth/overview).

When you receive the `user` object [on the client or the server](/docs/auth/overview#accessing-the-logged-in-user), you'll be able to access the user's GitHub ID like this:

```
    const githubIdentity = user.identities.github// GitHub User ID for example "12345678"githubIdentity.id
```

Read more about accessing the user data in the [Accessing User Data](/docs/auth/entities#accessing-the-auth-fields) section of the docs.

## API Reference[​](#api-reference "Direct link to API Reference")

Provider-specific behavior comes down to implementing two functions.

-   `configFn`
-   `userSignupFields`

The reference shows how to define both.

For behavior common to all providers, check the general [API Reference](/docs/auth/overview#api-reference).

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      gitHub: {        configFn: import { getConfig } from "@src/auth/github.js",        userSignupFields: import { userSignupFields } from "@src/auth/github.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      gitHub: {        configFn: import { getConfig } from "@src/auth/github.js",        userSignupFields: import { userSignupFields } from "@src/auth/github.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

The `gitHub` dict has the following properties:

-   #### `configFn: ExtImport`[​](#configfn-extimport "Direct link to configfn-extimport")
    
    This function should return an object with the scopes for the OAuth provider.
    
    -   JavaScript
    -   TypeScript
    
    src/auth/github.js
    
    ```
        export function getConfig() {  return {    scopes: [],  }}
    ```
    
    src/auth/github.ts
    
    ```
        export function getConfig() {  return {    scopes: [],  }}
    ```
    
-   #### `userSignupFields: ExtImport`[​](#usersignupfields-extimport "Direct link to usersignupfields-extimport")
    
    `userSignupFields` defines all the extra fields that need to be set on the `User` during the sign-up process. For example, if you have `address` and `phone` fields on your `User` entity, you can set them by defining the `userSignupFields` like this:
    
    -   JavaScript
    -   TypeScript
    
    src/auth.js
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    src/auth.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    Read more about the `userSignupFields` function [here](/docs/auth/overview#1-defining-extra-fields).

---

# Google

Wasp supports Google Authentication out of the box. Google Auth is arguably the best external auth option, as most users on the web already have Google accounts.

Enabling it lets your users log in using their existing Google accounts, greatly simplifying the process and enhancing the user experience.

Let's walk through enabling Google authentication, explain some of the default settings, and show how to override them.

## Setting up Google Auth[​](#setting-up-google-auth "Direct link to Setting up Google Auth")

Enabling Google Authentication comes down to a series of steps:

1.  Enabling Google authentication in the Wasp file.
2.  Adding the `User` entity.
3.  Creating a Google OAuth app.
4.  Adding the necessary Routes and Pages
5.  Using Auth UI components in our Pages.

Here's a skeleton of how our `main.wasp` should look like after we're done:

main.wasp

```
    // Configuring the social authenticationapp myApp {  auth: { ... }}// Defining routes and pagesroute LoginRoute { ... }page LoginPage { ... }
```

### 1\. Adding Google Auth to Your Wasp File[​](#1-adding-google-auth-to-your-wasp-file "Direct link to 1. Adding Google Auth to Your Wasp File")

Let's start by properly configuring the Auth object:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the User entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Google Auth      google: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the User entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Google Auth      google: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

`userEntity` is explained in [the social auth overview](/docs/auth/social-auth/overview#social-login-entity).

### 2\. Adding the User Entity[​](#2-adding-the-user-entity "Direct link to 2. Adding the User Entity")

Let's now define the `app.auth.userEntity` entity in the `schema.prisma` file:

-   JavaScript
-   TypeScript

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

### 3\. Creating a Google OAuth App[​](#3-creating-a-google-oauth-app "Direct link to 3. Creating a Google OAuth App")

To use Google as an authentication method, you'll first need to create a Google project and provide Wasp with your client key and secret. Here's how you do it:

1.  Create a Google Cloud Platform account if you do not already have one: [https://cloud.google.com/](https://cloud.google.com/)
2.  Create and configure a new Google project here: [https://console.cloud.google.com/home/dashboard](https://console.cloud.google.com/home/dashboard)

![Google Console Screenshot 1](/assets/images/integrations-google-1-e2b7988470aeea737ce29aeea573fbad.jpg)

![Google Console Screenshot 2](/assets/images/integrations-google-2-7e4bd9295f260173ea1a0f8a5177ac74.jpg)

3.  Search for **OAuth** in the top bar, click on **OAuth consent screen**.

![Google Console Screenshot 3](/assets/images/integrations-google-3-19147010e16baf7e68c86e84aebec158.jpg)

-   Select what type of app you want, we will go with **External**.
    
    ![Google Console Screenshot 4](/assets/images/integrations-google-4-7f063b6cbb9da5ebff47a5e34ef602b7.jpg)
    
-   Fill out applicable information on Page 1.
    
    ![Google Console Screenshot 5](/assets/images/integrations-google-5-3b7b41e3fd443156065fbf570f255d84.jpg)
    
-   On Page 2, Scopes, you should select `userinfo.profile`. You can optionally search for other things, like `email`.
    
    ![Google Console Screenshot 6](/assets/images/integrations-google-6-bc415d69af6b171e121a32c181bd1a45.jpg)
    
    ![Google Console Screenshot 7](/assets/images/integrations-google-7-a6f72bb8d3c88ce03c9d14708a6e5334.jpg)
    
    ![Google Console Screenshot 8](/assets/images/integrations-google-8-7dac6d0717e61d3794d22d2d7081ab72.jpg)
    
-   Add any test users you want on Page 3.
    
    ![Google Console Screenshot 9](/assets/images/integrations-google-9-eae399fe308c4661d0d1ab184d72f54c.jpg)
    

4.  Next, click **Credentials**.

![Google Console Screenshot 10](/assets/images/integrations-google-10-fbcac92ce4b2f0eadbbf2b6aed7a8b6b.jpg)

-   Select **Create Credentials**.
    
-   Select **OAuth client ID**.
    
    ![Google Console Screenshot 11](/assets/images/integrations-google-11-732ab31edf7887dde031e1db69762a8c.jpg)
    
-   Complete the form
    
    ![Google Console Screenshot 12](/assets/images/integrations-google-12-49c63cc46853b594eaf83d1ef918fe7e.jpg)
    
-   Under Authorized redirect URIs, put in: `http://localhost:3001/auth/google/callback`
    
    ![Google Console Screenshot 13](/assets/images/integrations-google-13-e2516dea25af3c8c37475f077ecf7368.jpg)
    
    -   Once you know on which URL(s) your API server will be deployed, also add those URL(s).
        -   For example: `https://your-server-url.com/auth/google/callback`
-   When you save, you can click the Edit icon and your credentials will be shown.
    
    ![Google Console Screenshot 14](/assets/images/integrations-google-14-ba3f829c32d4e8bd8429662add56a5b6.jpg)
    

5.  Copy your Client ID and Client secret as you will need them in the next step.

### 4\. Adding Environment Variables[​](#4-adding-environment-variables "Direct link to 4. Adding Environment Variables")

Add these environment variables to the `.env.server` file at the root of your project (take their values from the previous step):

.env.server

```
    GOOGLE_CLIENT_ID=your-google-client-idGOOGLE_CLIENT_SECRET=your-google-client-secret
```

### 5\. Adding the Necessary Routes and Pages[​](#5-adding-the-necessary-routes-and-pages "Direct link to 5. Adding the Necessary Routes and Pages")

Let's define the necessary authentication Routes and Pages.

Add the following code to your `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.jsx"}
```

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.tsx"}
```

We'll define the React components for these pages in the `src/pages/auth.tsx` file below.

### 6\. Create the Client Pages[​](#6-create-the-client-pages "Direct link to 6. Create the Client Pages")

info

We are using [Tailwind CSS](https://tailwindcss.com/) to style the pages. Read more about how to add it [here](/docs/project/css-frameworks).

Let's now create a `auth.tsx` file in the `src/pages`. It should have the following code:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

src/pages/auth.tsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }: { children: React.ReactNode }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

Auth UI

Our pages use an automatically-generated Auth UI component. Read more about Auth UI components [here](/docs/auth/ui).

### Conclusion[​](#conclusion "Direct link to Conclusion")

Yay, we've successfully set up Google Auth! 🎉

![Google Auth](/assets/images/google-1b331473c46517711eeaa5695f2fcb63.png)

Running `wasp db migrate-dev` and `wasp start` should now give you a working app with authentication. To see how to protect specific pages (i.e., hide them from non-authenticated users), read the docs on [using auth](/docs/auth/overview).

## Default Behaviour[​](#default-behaviour "Direct link to Default Behaviour")

Add `google: {}` to the `auth.methods` dictionary to use it with default settings:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

When a user **signs in for the first time**, Wasp creates a new user account and links it to the chosen auth provider account for future logins.

## Overrides[​](#overrides "Direct link to Overrides")

By default, Wasp doesn't store any information it receives from the social login provider. It only stores the user's ID specific to the provider.

There are two mechanisms used for overriding the default behavior:

-   `userSignupFields`
-   `configFn`

Let's explore them in more detail.

### Data Received From Google[​](#data-received-from-google "Direct link to Data Received From Google")

We are using Google's API and its `/userinfo` endpoint to fetch the user's data.

The data received from Google is an object which can contain the following fields:

```
    [  "name",  "given_name",  "family_name",  "email",  "email_verified",  "aud",  "exp",  "iat",  "iss",  "locale",  "picture",  "sub"]
```

The fields you receive depend on the scopes you request. The default scope is set to `profile` only. If you want to get the user's email, you need to specify the `email` scope in the `configFn` function.

For an up to date info about the data received from Google, please refer to the [Google API documentation](https://developers.google.com/identity/openid-connect/openid-connect#an-id-tokens-payload).

### Using the Data Received From Google[​](#using-the-data-received-from-google "Direct link to Using the Data Received From Google")

When a user logs in using a social login provider, the backend receives some data about the user. Wasp lets you access this data inside the `userSignupFields` getters.

For example, the User entity can include a `displayName` field which you can set based on the details received from the provider.

Wasp also lets you customize the configuration of the providers' settings using the `configFn` function.

Let's use this example to show both fields in action:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {        configFn: import { getConfig } from "@src/auth/google.js",        userSignupFields: import { userSignupFields } from "@src/auth/google.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/google.js

```
    export const userSignupFields = {  username: () => "hardcoded-username",  displayName: (data) => data.profile.name,}export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {        configFn: import { getConfig } from "@src/auth/google.js",        userSignupFields: import { userSignupFields } from "@src/auth/google.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/google.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  username: () => "hardcoded-username",  displayName: (data: any) => data.profile.name,})export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
```

Wasp automatically generates the `defineUserSignupFields` function to help you correctly type your `userSignupFields` object.

## Using Auth[​](#using-auth "Direct link to Using Auth")

To read more about how to set up the logout button and get access to the logged-in user in both client and server code, read the docs on [using auth](/docs/auth/overview).

When you receive the `user` object [on the client or the server](/docs/auth/overview#accessing-the-logged-in-user), you'll be able to access the user's Google ID like this:

```
    const googleIdentity = user.identities.google// Google User ID for example "123456789012345678901"googleIdentity.id
```

Read more about accessing the user data in the [Accessing User Data](/docs/auth/entities#accessing-the-auth-fields) section of the docs.

## API Reference[​](#api-reference "Direct link to API Reference")

Provider-specific behavior comes down to implementing two functions.

-   `configFn`
-   `userSignupFields`

The reference shows how to define both.

For behavior common to all providers, check the general [API Reference](/docs/auth/overview#api-reference).

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {        configFn: import { getConfig } from "@src/auth/google.js",        userSignupFields: import { userSignupFields } from "@src/auth/google.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      google: {        configFn: import { getConfig } from "@src/auth/google.js",        userSignupFields: import { userSignupFields } from "@src/auth/google.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

The `google` dict has the following properties:

-   #### `configFn: ExtImport`[​](#configfn-extimport "Direct link to configfn-extimport")
    
    This function must return an object with the scopes for the OAuth provider.
    
    -   JavaScript
    -   TypeScript
    
    src/auth/google.js
    
    ```
        export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
    ```
    
    src/auth/google.ts
    
    ```
        export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
    ```
    
-   #### `userSignupFields: ExtImport`[​](#usersignupfields-extimport "Direct link to usersignupfields-extimport")
    
    `userSignupFields` defines all the extra fields that need to be set on the `User` during the sign-up process. For example, if you have `address` and `phone` fields on your `User` entity, you can set them by defining the `userSignupFields` like this:
    
    -   JavaScript
    -   TypeScript
    
    src/auth.js
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    src/auth.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    Read more about the `userSignupFields` function [here](/docs/auth/overview#1-defining-extra-fields).

---

# Keycloak

Wasp supports Keycloak Authentication out of the box.

[Keycloak](https://www.keycloak.org/) is an open-source identity and access management solution for modern applications and services. Keycloak provides both SAML and OpenID protocol solutions. It also has a very flexible and powerful administration UI.

Let's walk through enabling Keycloak authentication, explain some of the default settings, and show how to override them.

## Setting up Keycloak Auth[​](#setting-up-keycloak-auth "Direct link to Setting up Keycloak Auth")

Enabling Keycloak Authentication comes down to a series of steps:

1.  Enabling Keycloak authentication in the Wasp file.
2.  Adding the `User` entity.
3.  Creating a Keycloak client.
4.  Adding the necessary Routes and Pages
5.  Using Auth UI components in our Pages.

Here's a skeleton of how our `main.wasp` should look like after we're done:

main.wasp

```
    // Configuring the social authenticationapp myApp {  auth: { ... }}// Defining routes and pagesroute LoginRoute { ... }page LoginPage { ... }
```

### 1\. Adding Keycloak Auth to Your Wasp File[​](#1-adding-keycloak-auth-to-your-wasp-file "Direct link to 1. Adding Keycloak Auth to Your Wasp File")

Let's start by properly configuring the Auth object:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the User entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Keycloak Auth      keycloak: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    // 1. Specify the User entity (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Keycloak Auth      keycloak: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

The `userEntity` is explained in [the social auth overview](/docs/auth/social-auth/overview#social-login-entity).

### 2\. Adding the User Entity[​](#2-adding-the-user-entity "Direct link to 2. Adding the User Entity")

Let's now define the `app.auth.userEntity` entity in the `schema.prisma` file:

-   JavaScript
-   TypeScript

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

### 3\. Creating a Keycloak Client[​](#3-creating-a-keycloak-client "Direct link to 3. Creating a Keycloak Client")

1.  Log into your Keycloak admin console.
    
2.  Under **Clients**, click on **Create Client**.
    
    ![Keycloak Screenshot 1](/assets/images/1-keycloak-8155a8f9942220fefbd9b9dcc2cad35a.png)
    
3.  Fill in the **Client ID** and choose a name for the client.
    
    ![Keycloak Screenshot 2](/assets/images/2-keycloak-32ad48fb343615017b70bd1eded25cb5.png)
    
4.  In the next step, enable **Client Authentication**.
    
    ![Keycloak Screenshot 3](/assets/images/3-keycloak-c0c90613d7359d7a25b4b658985fd86d.png)
    
5.  Under **Valid Redirect URIs**, add `http://localhost:3001/auth/keycloak/callback` for local development.
    
    ![Keycloak Screenshot 4](/assets/images/4-keycloak-8820ce6352eaa10525119c5aa363350c.png)
    
    -   Once you know on which URL(s) your API server will be deployed, also add those URL(s).
    -   For example: `https://my-server-url.com/auth/keycloak/callback`.
6.  Click **Save**.
    
7.  In the **Credentials** tab, copy the **Client Secret** value, which we'll use in the next step.
    
    ![Keycloak Screenshot 5](/assets/images/5-keycloak-bc37d067ab40972cae964de71798ede4.png)
    

### 4\. Adding Environment Variables[​](#4-adding-environment-variables "Direct link to 4. Adding Environment Variables")

Add these environment variables to the `.env.server` file at the root of your project (take their values from the previous step):

.env.server

```
    KEYCLOAK_CLIENT_ID=your-keycloak-client-idKEYCLOAK_CLIENT_SECRET=your-keycloak-client-secretKEYCLOAK_REALM_URL=https://your-keycloak-url.com/realms/master
```

We assumed in the `KEYCLOAK_REALM_URL` env variable that you are using the `master` realm. If you are using a different realm, replace `master` with your realm name.

### 5\. Adding the Necessary Routes and Pages[​](#5-adding-the-necessary-routes-and-pages "Direct link to 5. Adding the Necessary Routes and Pages")

Let's define the necessary authentication Routes and Pages.

Add the following code to your `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.jsx"}
```

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.tsx"}
```

We'll define the React components for these pages in the `src/pages/auth.tsx` file below.

### 6\. Create the Client Pages[​](#6-create-the-client-pages "Direct link to 6. Create the Client Pages")

info

We are using [Tailwind CSS](https://tailwindcss.com/) to style the pages. Read more about how to add it [here](/docs/project/css-frameworks).

Let's now create an `auth.tsx` file in the `src/pages`. It should have the following code:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

src/pages/auth.tsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }: { children: React.ReactNode }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

Auth UI

Our pages use an automatically generated Auth UI component. Read more about Auth UI components [here](/docs/auth/ui).

### Conclusion[​](#conclusion "Direct link to Conclusion")

Yay, we've successfully set up Keycloak Auth!

Running `wasp db migrate-dev` and `wasp start` should now give you a working app with authentication. To see how to protect specific pages (i.e., hide them from non-authenticated users), read the docs on [using auth](/docs/auth/overview).

## Default Behaviour[​](#default-behaviour "Direct link to Default Behaviour")

Add `keycloak: {}` to the `auth.methods` dictionary to use it with default settings:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      keycloak: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      keycloak: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

When a user **signs in for the first time**, Wasp creates a new user account and links it to the chosen auth provider account for future logins.

## Overrides[​](#overrides "Direct link to Overrides")

By default, Wasp doesn't store any information it receives from the social login provider. It only stores the user's ID specific to the provider.

There are two mechanisms used for overriding the default behavior:

-   `userSignupFields`
-   `configFn`

Let's explore them in more detail.

### Data Received From Keycloak[​](#data-received-from-keycloak "Direct link to Data Received From Keycloak")

We are using Keycloak's API and its `/userinfo` endpoint to fetch the user's data.

Keycloak user data

```
    {  sub: '5adba8fc-3ea6-445a-a379-13f0bb0b6969',  email_verified: true,  name: 'Test User',  preferred_username: 'test',  given_name: 'Test',  family_name: 'User',  email: 'test@example.com'}
```

The fields you receive will depend on the scopes you requested. The default scope is set to `profile` only. If you want to get the user's email, you need to specify the `email` scope in the `configFn` function.

For up-to-date info about the data received from Keycloak, please refer to the [Keycloak API documentation](https://www.keycloak.org/docs-api/23.0.7/javadocs/org/keycloak/representations/UserInfo.html).

### Using the Data Received From Keycloak[​](#using-the-data-received-from-keycloak "Direct link to Using the Data Received From Keycloak")

When a user logs in using a social login provider, the backend receives some data about the user. Wasp lets you access this data inside the `userSignupFields` getters.

For example, the User entity can include a `displayName` field which you can set based on the details received from the provider.

Wasp also lets you customize the configuration of the providers' settings using the `configFn` function.

Let's use this example to show both fields in action:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      keycloak: {        configFn: import { getConfig } from "@src/auth/keycloak.js",        userSignupFields: import { userSignupFields } from "@src/auth/keycloak.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/keycloak.js

```
    export const userSignupFields = {  username: () => "hardcoded-username",  displayName: (data) => data.profile.name,}export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      keycloak: {        configFn: import { getConfig } from "@src/auth/keycloak.js",        userSignupFields: import { userSignupFields } from "@src/auth/keycloak.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/keycloak.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  username: () => "hardcoded-username",  displayName: (data: any) => data.profile.name,})export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
```

Wasp automatically generates the `defineUserSignupFields` function to help you correctly type your `userSignupFields` object.

## Using Auth[​](#using-auth "Direct link to Using Auth")

To read more about how to set up the logout button and get access to the logged-in user in both client and server code, read the docs on [using auth](/docs/auth/overview).

When you receive the `user` object [on the client or the server](/docs/auth/overview#accessing-the-logged-in-user), you'll be able to access the user's Keycloak ID like this:

```
    const keycloakIdentity = user.identities.keycloak// Keycloak User ID for example "12345678-1234-1234-1234-123456789012"keycloakIdentity.id
```

Read more about accessing the user data in the [Accessing User Data](/docs/auth/entities#accessing-the-auth-fields) section of the docs.

## API Reference[​](#api-reference "Direct link to API Reference")

Provider-specific behavior comes down to implementing two functions.

-   `configFn`
-   `userSignupFields`

The reference shows how to define both.

For behavior common to all providers, check the general [API Reference](/docs/auth/overview#api-reference).

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      keycloak: {        configFn: import { getConfig } from "@src/auth/keycloak.js",        userSignupFields: import { userSignupFields } from "@src/auth/keycloak.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      keycloak: {        configFn: import { getConfig } from "@src/auth/keycloak.js",        userSignupFields: import { userSignupFields } from "@src/auth/keycloak.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

The `keycloak` dict has the following properties:

-   #### `configFn: ExtImport`[​](#configfn-extimport "Direct link to configfn-extimport")
    
    This function must return an object with the scopes for the OAuth provider.
    
    -   JavaScript
    -   TypeScript
    
    src/auth/keycloak.js
    
    ```
        export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
    ```
    
    src/auth/keycloak.ts
    
    ```
        export function getConfig() {  return {    scopes: ['profile', 'email'],  }}
    ```
    
-   #### `userSignupFields: ExtImport`[​](#usersignupfields-extimport "Direct link to usersignupfields-extimport")
    
    `userSignupFields` defines all the extra fields that need to be set on the `User` during the sign-up process. For example, if you have `address` and `phone` fields on your `User` entity, you can set them by defining the `userSignupFields` like this:
    
    -   JavaScript
    -   TypeScript
    
    src/auth.js
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    src/auth.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    Read more about the `userSignupFields` function [here](/docs/auth/overview#1-defining-extra-fields).

---

# Discord

Wasp supports Discord Authentication out of the box.

Letting your users log in using their Discord accounts turns the signup process into a breeze.

Let's walk through enabling Discord Authentication, explain some of the default settings, and show how to override them.

## Setting up Discord Auth[​](#setting-up-discord-auth "Direct link to Setting up Discord Auth")

Enabling Discord Authentication comes down to a series of steps:

1.  Enabling Discord authentication in the Wasp file.
2.  Adding the `User` entity.
3.  Creating a Discord App.
4.  Adding the necessary Routes and Pages
5.  Using Auth UI components in our Pages.

Here's a skeleton of how our `main.wasp` should look like after we're done:

main.wasp

```
    // Configuring the social authenticationapp myApp {  auth: { ... }}// Defining routes and pagesroute LoginRoute { ... }page LoginPage { ... }
```

### 1\. Adding Discord Auth to Your Wasp File[​](#1-adding-discord-auth-to-your-wasp-file "Direct link to 1. Adding Discord Auth to Your Wasp File")

Let's start by properly configuring the Auth object:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    // 1. Specify the User entity  (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Discord Auth      discord: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    // 1. Specify the User entity  (we'll define it next)    userEntity: User,    methods: {      // 2. Enable Discord Auth      discord: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

### 2\. Add the User Entity[​](#2-add-the-user-entity "Direct link to 2. Add the User Entity")

Let's now define the `app.auth.userEntity` entity in the `schema.prisma` file:

-   JavaScript
-   TypeScript

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

schema.prisma

```
    // 3. Define the user entitymodel User {  id Int @id @default(autoincrement())  // Add your own fields below  // ...}
```

### 3\. Creating a Discord App[​](#3-creating-a-discord-app "Direct link to 3. Creating a Discord App")

To use Discord as an authentication method, you'll first need to create a Discord App and provide Wasp with your client key and secret. Here's how you do it:

1.  Log into your Discord account and navigate to: [https://discord.com/developers/applications](https://discord.com/developers/applications).
2.  Select **New Application**.
3.  Supply required information.

![Discord Applications Screenshot](/img/integrations-discord-1.png)

4.  Go to the **OAuth2** tab on the sidebar and click **Add Redirect**

-   For development, put: `http://localhost:3001/auth/discord/callback`.
-   Once you know on which URL your API server will be deployed, you can create a new app with that URL instead e.g. `https://your-server-url.com/auth/discord/callback`.

4.  Hit **Save Changes**.
5.  Hit **Reset Secret**.
6.  Copy your Client ID and Client secret as you'll need them in the next step.

### 4\. Adding Environment Variables[​](#4-adding-environment-variables "Direct link to 4. Adding Environment Variables")

Add these environment variables to the `.env.server` file at the root of your project (take their values from the previous step):

.env.server

```
    DISCORD_CLIENT_ID=your-discord-client-idDISCORD_CLIENT_SECRET=your-discord-client-secret
```

### 5\. Adding the Necessary Routes and Pages[​](#5-adding-the-necessary-routes-and-pages "Direct link to 5. Adding the Necessary Routes and Pages")

Let's define the necessary authentication Routes and Pages.

Add the following code to your `main.wasp` file:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.jsx"}
```

main.wasp

```
    // ...route LoginRoute { path: "/login", to: LoginPage }page LoginPage {  component: import { Login } from "@src/pages/auth.tsx"}
```

We'll define the React components for these pages in the `src/pages/auth.tsx` file below.

### 6\. Creating the Client Pages[​](#6-creating-the-client-pages "Direct link to 6. Creating the Client Pages")

info

We are using [Tailwind CSS](https://tailwindcss.com/) to style the pages. Read more about how to add it [here](/docs/project/css-frameworks).

Let's create a `auth.tsx` file in the `src/pages` folder and add the following to it:

-   JavaScript
-   TypeScript

src/pages/auth.jsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

src/pages/auth.tsx

```
    import { LoginForm } from 'wasp/client/auth'export function Login() {  return (    <Layout>      <LoginForm />    </Layout>  )}// A layout component to center the contentexport function Layout({ children }: { children: React.ReactNode }) {  return (    <div className="w-full h-full bg-white">      <div className="min-w-full min-h-[75vh] flex items-center justify-center">        <div className="w-full h-full max-w-sm p-5 bg-white">          <div>{children}</div>        </div>      </div>    </div>  )}
```

We imported the generated Auth UI components and used them in our pages. Read more about the Auth UI components [here](/docs/auth/ui).

### Conclusion[​](#conclusion "Direct link to Conclusion")

Yay, we've successfully set up Discord Auth! 🎉

![Discord Auth](/assets/images/discord-584bbfefb7948f2a3b73bb30adb0346c.png)

Running `wasp db migrate-dev` and `wasp start` should now give you a working app with authentication. To see how to protect specific pages (i.e., hide them from non-authenticated users), read the docs on [using auth](/docs/auth/overview).

## Default Behaviour[​](#default-behaviour "Direct link to Default Behaviour")

Add `discord: {}` to the `auth.methods` dictionary to use it with default settings.

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      discord: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      discord: {}    },    onAuthFailedRedirectTo: "/login"  },}
```

When a user **signs in for the first time**, Wasp creates a new user account and links it to the chosen auth provider account for future logins.

## Overrides[​](#overrides "Direct link to Overrides")

By default, Wasp doesn't store any information it receives from the social login provider. It only stores the user's ID specific to the provider.

There are two mechanisms used for overriding the default behavior:

-   `userSignupFields`
-   `configFn`

Let's explore them in more detail.

### Data Received From Discord[​](#data-received-from-discord "Direct link to Data Received From Discord")

We are using Discord's API and its `/users/@me` endpoint to get the user data.

The data we receive from Discord on the `/users/@me` endpoint looks something like this:

```
    {  "id": "80351110224678912",  "username": "Nelly",  "discriminator": "1337",  "avatar": "8342729096ea3675442027381ff50dfe",  "verified": true,  "flags": 64,  "banner": "06c16474723fe537c283b8efa61a30c8",  "accent_color": 16711680,  "premium_type": 1,  "public_flags": 64,  "avatar_decoration_data": {    "sku_id": "1144058844004233369",    "asset": "a_fed43ab12698df65902ba06727e20c0e"  }}
```

The fields you receive will depend on the scopes you requested. The default scope is set to `identify` only. If you want to get the email, you need to specify the `email` scope in the `configFn` function.

For an up to date info about the data received from Discord, please refer to the [Discord API documentation](https://discord.com/developers/docs/resources/user#user-object-user-structure).

### Using the Data Received From Discord[​](#using-the-data-received-from-discord "Direct link to Using the Data Received From Discord")

When a user logs in using a social login provider, the backend receives some data about the user. Wasp lets you access this data inside the `userSignupFields` getters.

For example, the User entity can include a `displayName` field which you can set based on the details received from the provider.

Wasp also lets you customize the configuration of the providers' settings using the `configFn` function.

Let's use this example to show both fields in action:

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      discord: {        configFn: import { getConfig } from "@src/auth/discord.js",        userSignupFields: import { userSignupFields } from "@src/auth/discord.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/discord.js

```
    export const userSignupFields = {  username: (data) => data.profile.global_name,  avatarUrl: (data) => data.profile.avatar,};export function getConfig() {  return {    scopes: ['identify'],  };}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      discord: {        configFn: import { getConfig } from "@src/auth/discord.js",        userSignupFields: import { userSignupFields } from "@src/auth/discord.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

schema.prisma

```
    model User {  id          Int    @id @default(autoincrement())  username    String @unique  displayName String}// ...
```

src/auth/discord.ts

```
    import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  username: (data: any) => data.profile.global_name,  avatarUrl: (data: any) => data.profile.avatar,})export function getConfig() {  return {    scopes: ['identify'],  }}
```

Wasp automatically generates the `defineUserSignupFields` function to help you correctly type your `userSignupFields` object.

## Using Auth[​](#using-auth "Direct link to Using Auth")

To read more about how to set up the logout button and get access to the logged-in user in both client and server code, read the docs on [using auth](/docs/auth/overview).

When you receive the `user` object [on the client or the server](/docs/auth/overview#accessing-the-logged-in-user), you'll be able to access the user's Discord ID like this:

```
    const discordIdentity = user.identities.discord// Discord User ID for example "80351110224678912"discordIdentity.id
```

Read more about accessing the user data in the [Accessing User Data](/docs/auth/entities#accessing-the-auth-fields) section of the docs.

## API Reference[​](#api-reference "Direct link to API Reference")

Provider-specific behavior comes down to implementing two functions.

-   `configFn`
-   `userSignupFields`

The reference shows how to define both.

For behavior common to all providers, check the general [API Reference](/docs/auth/overview#api-reference).

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      discord: {        configFn: import { getConfig } from "@src/auth/discord.js",        userSignupFields: import { userSignupFields } from "@src/auth/discord.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      discord: {        configFn: import { getConfig } from "@src/auth/discord.js",        userSignupFields: import { userSignupFields } from "@src/auth/discord.js"      }    },    onAuthFailedRedirectTo: "/login"  },}
```

The `discord` dict has the following properties:

-   #### `configFn: ExtImport`[​](#configfn-extimport "Direct link to configfn-extimport")
    
    This function should return an object with the scopes for the OAuth provider.
    
    -   JavaScript
    -   TypeScript
    
    src/auth/discord.js
    
    ```
        export function getConfig() {  return {    scopes: [],  }}
    ```
    
    src/auth/discord.ts
    
    ```
        export function getConfig() {  return {    scopes: [],  }}
    ```
    
-   #### `userSignupFields: ExtImport`[​](#usersignupfields-extimport "Direct link to usersignupfields-extimport")
    
    `userSignupFields` defines all the extra fields that need to be set on the `User` during the sign-up process. For example, if you have `address` and `phone` fields on your `User` entity, you can set them by defining the `userSignupFields` like this:
    
    -   JavaScript
    -   TypeScript
    
    src/auth.js
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    src/auth.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({  address: (data) => {    if (!data.address) {      throw new Error('Address is required')    }    return data.address  }  phone: (data) => data.phone,})
    ```
    
    Read more about the `userSignupFields` function [here](/docs/auth/overview#1-defining-extra-fields).

---

# Accessing User Data

First, we'll check out the most practical info: **how to access the user's data in your app**.

Then, we'll dive into the details of the **auth entities** that Wasp creates behind the scenes to store the user's data. For auth each method, Wasp needs to store different information about the user. For example, username for [Username & password](/docs/auth/username-and-pass) auth, email verification status for [Email](/docs/auth/email) auth, and so on.

We'll also show you how you can use these entities to create a custom signup action.

## Accessing the Auth Fields[​](#accessing-the-auth-fields "Direct link to Accessing the Auth Fields")

When you receive the `user` object [on the client or the server](/docs/auth/overview#accessing-the-logged-in-user), it will contain all the user fields you defined in the `User` entity in the `schema.prisma` file. In addition to that, it will also contain all the auth-related fields that Wasp stores. This includes things like the `username` or the email verification status. In Wasp, this data is called the `AuthUser` object.

### `AuthUser` Object Fields[​](#authuser-object-fields "Direct link to authuser-object-fields")

All the `User` fields you defined will be present at the top level of the `AuthUser` object. The auth-related fields will be on the `identities` object. For each auth method you enable, there will be a separate data object in the `identities` object.

The `AuthUser` object will change depending on which auth method you have enabled in the Wasp file. For example, if you enabled the email auth and Google auth, it would look something like this:

-   User Signed Up with Google
-   User Signed Up with Email

If the user has only the Google identity, the `AuthUser` object will look like this:

```
    const user = {  // User data  id: 'cluqs9qyh00007cn73apj4hp7',  address: 'Some address',  // Auth methods specific data  identities: {    email: null,    google: {      id: '1117XXXX1301972049448',    },  },}
```

If the user has only the email identity, the `AuthUser` object will look like this:

```
    const user = {  // User data  id: 'cluqsex9500017cn7i2hwsg17',  address: 'Some address',  // Auth methods specific data  identities: {    email: {      id: 'user@app.com',      isEmailVerified: true,      emailVerificationSentAt: '2024-04-08T10:06:02.204Z',      passwordResetSentAt: null,    },    google: null,  },}
```

In the examples above, you can see the `identities` object contains the `email` and `google` objects. The `email` object contains the email-related data and the `google` object contains the Google-related data.

Make sure to check if the data exists

Before accessing some auth method's data, you'll need to check if that data exists for the user and then access it:

```
    if (user.identities.google !== null) {  const userId = user.identities.google.id  // ...}
```

You need to do this because if a user didn't sign up with some auth method, the data for that auth method will be `null`.

Let's look at the data for each of the available auth methods:

-   [Username & password](/docs/auth/username-and-pass) data
    
    ```
        const usernameIdentity = user.identities.username// Username that the user used to sign up, e.g. "fluffyllama"usernameIdentity.id
    ```
    
-   [Email](/docs/auth/email) data
    
    ```
        const emailIdentity = user.identities.email// Email address the the user used to sign up, e.g. "fluffyllama@app.com".emailIdentity.id// `true` if the user has verified their email address.emailIdentity.isEmailVerified// Datetime when the email verification email was sent.emailIdentity.emailVerificationSentAt// Datetime when the last password reset email was sent.emailIdentity.passwordResetSentAt
    ```
    
-   [Google](/docs/auth/social-auth/google) data
    
    ```
        const googleIdentity = user.identities.google// Google User ID for example "123456789012345678901"googleIdentity.id
    ```
    
-   [GitHub](/docs/auth/social-auth/github) data
    
    ```
        const githubIdentity = user.identities.github// GitHub User ID for example "12345678"githubIdentity.id
    ```
    
-   [Keycloak](/docs/auth/social-auth/keycloak) data
    
    ```
        const keycloakIdentity = user.identities.keycloak// Keycloak User ID for example "12345678-1234-1234-1234-123456789012"keycloakIdentity.id
    ```
    
-   [Discord](/docs/auth/social-auth/discord) data
    
    ```
        const discordIdentity = user.identities.discord// Discord User ID for example "80351110224678912"discordIdentity.id
    ```
    

If you support multiple auth methods, you'll need to find which identity exists for the user and then access its data:

```
    if (user.identities.email !== null) {  const email = user.identities.email.id  // ...} else if (user.identities.google !== null) {  const googleId = user.identities.google.id  // ...}
```

### `getFirstProviderUserId` Helper[​](#getfirstprovideruserid-helper "Direct link to getfirstprovideruserid-helper")

The `getFirstProviderUserId` method returns the first user ID that it finds for the user. For example if the user has signed up with email, it will return the email. If the user has signed up with Google, it will return the Google ID.

This can be useful if you support multiple authentication methods and you need *any* ID that identifies the user in your app.

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    const MainPage = ({ user }) => {  const userId = user.getFirstProviderUserId()  // ...}
```

src/tasks.js

```
    export const createTask = async (args, context) => {  const userId = context.user.getFirstProviderUserId()  // ...}
```

src/MainPage.tsx

```
    import { type AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const userId = user.getFirstProviderUserId()  // ...}
```

src/tasks.ts

```
    export const createTask: CreateTask<...>  = async (args, context) => {  const userId = context.user.getFirstProviderUserId()  // ...}
```

\* Multiple identities per user will be possible in the future and then the `getFirstProviderUserId` method will return the ID of the first identity that it finds without any guarantees about which one it will be.

## Including the User with Other Entities[​](#including-the-user-with-other-entities "Direct link to Including the User with Other Entities")

Sometimes, you might want to include the user's data when fetching other entities. For example, you might want to include the user's data with the tasks they have created.

We'll mention the `auth` and the `identities` relations which we will explain in more detail later in the [Entities Explained](#entities-explained) section.

Be careful about sensitive data

You'll need to include the `auth` and the `identities` relations to get the full auth data about the user. However, you should keep in mind that the `providerData` field in the `identities` can contain sensitive data like the user's hashed password (in case of email or username auth), so you will likely want to exclude it if you are returning those values to the client.

You can include the full user's data with other entities using the `include` option in the Prisma queries:

-   JavaScript
-   TypeScript

src/tasks.js

```
    export const getAllTasks = async (args, context) => {  return context.entities.Task.findMany({    orderBy: { id: 'desc' },    select: {      id: true,      title: true,      user: {        include: {          auth: {            include: {              identities: {                // Including only the `providerName` and `providerUserId` fields                select: {                  providerName: true,                  providerUserId: true,                },              },            },          },        },      },    },  })}
```

src/tasks.ts

```
    export const getAllTasks = (async (args, context) => {  return context.entities.Task.findMany({    orderBy: { id: 'desc' },    select: {      id: true,      title: true,      user: {        include: {          auth: {            include: {              identities: {                // Including only the `providerName` and `providerUserId` fields                select: {                  providerName: true,                  providerUserId: true,                },              },            },          },        },      },    },  })}) satisfies tasks.GetAllQuery<{}, {}>
```

If you have some **piece of the auth data that you want to access frequently** (for example the `username`), it's best to store it at the top level of the `User` entity.

For example, save the `username` or `email` as a property on the `User` and you'll be able to access it without including the `auth` and `identities` fields. We show an example in the [Defining Extra Fields on the User Entity](/docs/auth/overview#1-defining-extra-fields) section of the docs.

### Getting Auth Data from the User Object[​](#getting-auth-data-from-the-user-object "Direct link to Getting Auth Data from the User Object")

When you have the `user` object with the `auth` and `identities` fields, it can be a bit tedious to obtain the auth data (like username or Google ID) from it:

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {task.user.auth?.identities[0].providerUserId}        </div>      ))}    </div>  )}
```

src/MainPage.tsx

```
    function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {task.user.auth?.identities[0].providerUserId}        </div>      ))}    </div>  )}
```

Wasp offers a few helper methods to access the user's auth data when you retrieve the `user` like this. They are `getUsername`, `getEmail` and `getFirstProviderUserId`. They can be used both on the client and the server.

#### `getUsername`[​](#getusername "Direct link to getusername")

It accepts the `user` object and if the user signed up with the [Username & password](/docs/auth/username-and-pass) auth method, it returns the username or `null` otherwise. The `user` object needs to have the `auth` and the `identities` relations included.

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import { getUsername } from 'wasp/auth'function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {getUsername(task.user)}        </div>      ))}    </div>  )}
```

src/MainPage.tsx

```
    import { getUsername } from 'wasp/auth'function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {getUsername(task.user)}        </div>      ))}    </div>  )}
```

#### `getEmail`[​](#getemail "Direct link to getemail")

It accepts the `user` object and if the user signed up with the [Email](/docs/auth/email) auth method, it returns the email or `null` otherwise. The `user` object needs to have the `auth` and the `identities` relations included.

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import { getEmail } from 'wasp/auth'function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {getEmail(task.user)}        </div>      ))}    </div>  )}
```

src/MainPage.tsx

```
    import { getEmail } from 'wasp/auth'function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {getEmail(task.user)}        </div>      ))}    </div>  )}
```

#### `getFirstProviderUserId`[​](#getfirstprovideruserid "Direct link to getfirstprovideruserid")

It returns the first user ID that it finds for the user. For example if the user has signed up with email, it will return the email. If the user has signed up with Google, it will return the Google ID. The `user` object needs to have the `auth` and the `identities` relations included.

-   JavaScript
-   TypeScript

src/MainPage.jsx

```
    import { getFirstProviderUserId } from 'wasp/auth'function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {getFirstProviderUserId(task.user)}        </div>      ))}    </div>  )}
```

src/MainPage.tsx

```
    import { getFirstProviderUserId } from 'wasp/auth'function MainPage() {  // ...  return (    <div className="tasks">      {tasks.map((task) => (        <div key={task.id} className="task">          {task.title} by {getFirstProviderUserId(task.user)}        </div>      ))}    </div>  )}
```

## Entities Explained[​](#entities-explained "Direct link to Entities Explained")

To store user's auth information, Wasp does a few things behind the scenes. Wasp takes your `schema.prisma` file and combines it with additional entities to create the final `schema.prisma` file that is used in your app.

In this section, we will explain which entities are created and how they are connected.

### User Entity[​](#user-entity "Direct link to User Entity")

When you want to add authentication to your app, you need to specify the `userEntity` field.

For example, you might set it to `User`:

main.wasp

```
    app myApp {  wasp: {    version: "^0.14.0"  },  title: "My App",  auth: {    userEntity: User,    // ...  },}
```

And define the `User` in the `schema.prisma` file:

schema.prisma

```
    model User {  id Int @id @default(autoincrement())  // Any other fields you want to store about the user}
```

The `User` entity is a "business logic user" which represents a user of your app.

You can use this entity to store any information about the user that you want to store. For example, you might want to store the user's name or address.

You can also use the user entity to define the relations between users and other entities in your app. For example, you might want to define a relation between a user and the tasks that they have created.

You **own** the user entity and you can modify it as you wish. You can add new fields to it, remove fields from it, or change the type of the fields. You can also add new relations to it or remove existing relations from it.

![Auth Entities in a Wasp App](/img/auth-entities/model.png)

Auth Entities in a Wasp App

On the other hand, the `Auth`, `AuthIdentity` and `Session` entities are created behind the scenes and are used to store the user's login credentials. You as the developer don't need to care about this entity most of the time. Wasp **owns** these entities.

In the case you want to create a custom signup action, you will need to use the `Auth` and `AuthIdentity` entities directly.

### Example App Model[​](#example-app-model "Direct link to Example App Model")

Let's imagine we created a simple tasks management app:

-   The app has email and Google-based auth.
-   Users can create tasks and see the tasks that they have created.

Let's look at how would that look in the database:

![Example of Auth Entities](/img/auth-entities/model-example.png)

Example of Auth Entities

If we take a look at an example user in the database, we can see:

-   The business logic user, `User` is connected to multiple `Task` entities.
    -   In this example, "Example User" has two tasks.
-   The `User` is connected to exactly one `Auth` entity.
-   Each `Auth` entity can have multiple `AuthIdentity` entities.
    -   In this example, the `Auth` entity has two `AuthIdentity` entities: one for the email-based auth and one for the Google-based auth.
-   Each `Auth` entity can have multiple `Session` entities.
    -   In this example, the `Auth` entity has one `Session` entity.

Using multiple auth identities for a single user

Wasp currently doesn't support multiple auth identities for a single user. This means, for example, that a user can't have both an email-based auth identity and a Google-based auth identity. This is something we will add in the future with the introduction of the [account merging feature](https://github.com/wasp-lang/wasp/issues/954).

Account merging means that multiple auth identities can be merged into a single user account. For example, a user's email and Google identity can be merged into a single user account. Then the user can log in with either their email or Google account and they will be logged into the same account.

### `Auth` Entity internal[​](#auth-entity- "Direct link to auth-entity-")

Wasp's internal `Auth` entity is used to connect the business logic user, `User` with the user's login credentials.

```
    model Auth {  id         String         @id @default(uuid())  userId     Int?           @unique  // Wasp injects this relation on the User entity as well  user       User?          @relation(fields: [userId], references: [id], onDelete: Cascade)  identities AuthIdentity[]  sessions   Session[]}
```

The `Auth` fields:

-   `id` is a unique identifier of the `Auth` entity.
-   `userId` is a foreign key to the `User` entity.
    -   It is used to connect the `Auth` entity with the business logic user.
-   `user` is a relation to the `User` entity.
    -   This relation is injected on the `User` entity as well.
-   `identities` is a relation to the `AuthIdentity` entity.
-   `sessions` is a relation to the `Session` entity.

### `AuthIdentity` Entity internal[​](#authidentity-entity- "Direct link to authidentity-entity-")

The `AuthIdentity` entity is used to store the user's login credentials for various authentication methods.

```
    model AuthIdentity {  providerName   String  providerUserId String  providerData   String @default("{}")  authId         String  auth           Auth   @relation(fields: [authId], references: [id], onDelete: Cascade)  @@id([providerName, providerUserId])}
```

The `AuthIdentity` fields:

-   `providerName` is the name of the authentication provider.
    -   For example, `email` or `google`.
-   `providerUserId` is the user's ID in the authentication provider.
    -   For example, the user's email or Google ID.
-   `providerData` is a JSON string that contains additional data about the user from the authentication provider.
    -   For example, for password based auth, this field contains the user's hashed password.
    -   This field is a `String` and not a `Json` type because [Prisma doesn't support the `Json` type for SQLite](https://github.com/prisma/prisma/issues/3786).
-   `authId` is a foreign key to the `Auth` entity.
    -   It is used to connect the `AuthIdentity` entity with the `Auth` entity.
-   `auth` is a relation to the `Auth` entity.

### `Session` Entity internal[​](#session-entity- "Direct link to session-entity-")

The `Session` entity is used to store the user's session information. It is used to keep the user logged in between page refreshes.

```
    model Session {  id        String   @id @unique  expiresAt DateTime  userId    String  auth      Auth     @relation(references: [id], fields: [userId], onDelete: Cascade)  @@index([userId])}
```

The `Session` fields:

-   `id` is a unique identifier of the `Session` entity.
-   `expiresAt` is the date when the session expires.
-   `userId` is a foreign key to the `Auth` entity.
    -   It is used to connect the `Session` entity with the `Auth` entity.
-   `auth` is a relation to the `Auth` entity.

## Custom Signup Action[​](#custom-signup-action "Direct link to Custom Signup Action")

Let's take a look at how you can use the `Auth` and `AuthIdentity` entities to create custom login and signup actions. For example, you might want to create a custom signup action that creates a user in your app and also creates a user in a third-party service.

Custom Signup Examples

In the [Email](/docs/auth/email#creating-a-custom-sign-up-action) section of the docs we give you an example for custom email signup and in the [Username & password](/docs/auth/username-and-pass#2-creating-your-custom-sign-up-action) section of the docs we give you an example for custom username & password signup.

Below is a simplified version of a custom signup action which you probably wouldn't use in your app but it shows you how you can use the `Auth` and `AuthIdentity` entities to create a custom signup action.

-   JavaScript
-   TypeScript

main.wasp

```
    // ...action customSignup {  fn: import { signup } from "@src/auth/signup.js",  entities: [User]}
```

src/auth/signup.js

```
    import {  createProviderId,  sanitizeAndSerializeProviderData,  createUser,} from 'wasp/server/auth'export const signup = async (args, { entities: { User } }) => {  try {    // Provider ID is a combination of the provider name and the provider user ID    // And it is used to uniquely identify the user in your app    const providerId = createProviderId('username', args.username)    // sanitizeAndSerializeProviderData hashes the password and returns a JSON string    const providerData = await sanitizeAndSerializeProviderData({      hashedPassword: args.password,    })    await createUser(      providerId,      providerData,      // Any additional data you want to store on the User entity      {}    )    // This is equivalent to:    // await User.create({    //   data: {    //     auth: {    //       create: {    //         identities: {    //             create: {    //                 providerName: 'username',    //                 providerUserId: args.username    //                 providerData,    //             },    //         },    //       }    //     },    //   }    // })  } catch (e) {    return {      success: false,      message: e.message,    }  }  // Your custom code after sign-up.  // ...  return {    success: true,    message: 'User created successfully',  }}
```

main.wasp

```
    // ...action customSignup {  fn: import { signup } from "@src/auth/signup.js",  entities: [User]}
```

src/auth/signup.ts

```
    import {  createProviderId,  sanitizeAndSerializeProviderData,  createUser,} from 'wasp/server/auth'import type { CustomSignup } from 'wasp/server/operations'type CustomSignupInput = {  username: string  password: string}type CustomSignupOutput = {  success: boolean  message: string}export const signup: CustomSignup<  CustomSignupInput,  CustomSignupOutput> = async (args, { entities: { User } }) => {  try {    // Provider ID is a combination of the provider name and the provider user ID    // And it is used to uniquely identify the user in your app    const providerId = createProviderId('username', args.username)    // sanitizeAndSerializeProviderData hashes the password and returns a JSON string    const providerData = await sanitizeAndSerializeProviderData<'username'>({      hashedPassword: args.password,    })    await createUser(      providerId,      providerData,      // Any additional data you want to store on the User entity      {}    )    // This is equivalent to:    // await User.create({    //   data: {    //     auth: {    //       create: {    //         identities: {    //             create: {    //                 providerName: 'username',    //                 providerUserId: args.username    //                 providerData,    //             },    //         },    //       }    //     },    //   }    // })  } catch (e) {    return {      success: false,      message: e.message,    }  }  // Your custom code after sign-up.  // ...  return {    success: true,    message: 'User created successfully',  }}
```

You can use whichever method suits your needs better: either the `createUser` function or Prisma's `User.create` method. The `createUser` function is a bit more convenient to use because it hides some of the complexity. On the other hand, the `User.create` method gives you more control over the data that is stored in the `Auth` and `AuthIdentity` entities.

---

# Auth Hooks

Auth hooks allow you to "hook into" the auth process at various stages and run your custom code. For example, if you want to forbid certain emails from signing up, or if you wish to send a welcome email to the user after they sign up, auth hooks are the way to go.

## Supported hooks[​](#supported-hooks "Direct link to Supported hooks")

The following auth hooks are available in Wasp:

-   [`onBeforeSignup`](#executing-code-before-the-user-signs-up)
-   [`onAfterSignup`](#executing-code-after-the-user-signs-up)
-   [`onBeforeOAuthRedirect`](#executing-code-before-the-oauth-redirect)

We'll go through each of these hooks in detail. But first, let's see how the hooks fit into the signup flow:

![Signup Flow with Hooks](/img/auth-hooks/signup_flow_with_hooks.png)

Signup Flow with Hooks

If you are using OAuth, the flow includes extra steps before the signup flow:

![OAuth Flow with Hooks](/img/auth-hooks/oauth_flow_with_hooks.png)

OAuth Flow with Hooks

## Using hooks[​](#using-hooks "Direct link to Using hooks")

To use auth hooks, you must first declare them in the Wasp file:

-   JavaScript
-   TypeScript

```
    app myApp {  wasp: {    version: "^0.13.0"  },  auth: {    userEntity: User,    methods: {      ...    },    onBeforeSignup: import { onBeforeSignup } from "@src/auth/hooks",    onAfterSignup: import { onAfterSignup } from "@src/auth/hooks",    onBeforeOAuthRedirect: import { onBeforeOAuthRedirect } from "@src/auth/hooks",  },}
```

```
    app myApp {  wasp: {    version: "^0.13.0"  },  auth: {    userEntity: User,    methods: {      ...    },    onBeforeSignup: import { onBeforeSignup } from "@src/auth/hooks",    onAfterSignup: import { onAfterSignup } from "@src/auth/hooks",    onBeforeOAuthRedirect: import { onBeforeOAuthRedirect } from "@src/auth/hooks",  },}
```

If the hooks are defined as async functions, Wasp *awaits* them. This means the auth process waits for the hooks to finish before continuing.

Wasp ignores the hooks' return values. The only exception is the `onBeforeOAuthRedirect` hook, whose return value affects the OAuth redirect URL.

We'll now go through each of the available hooks.

### Executing code before the user signs up[​](#executing-code-before-the-user-signs-up "Direct link to Executing code before the user signs up")

Wasp calls the `onBeforeSignup` hook before the user is created.

The `onBeforeSignup` hook can be useful if you want to reject a user based on some criteria before they sign up.

Works with [Email](/docs/auth/email) [Username & Password](/docs/auth/username-and-pass) [Discord](/docs/auth/social-auth/discord) [Github](/docs/auth/social-auth/github) [Google](/docs/auth/social-auth/google) [Keycloak](/docs/auth/social-auth/keycloak)

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  ...  auth: {    ...    onBeforeSignup: import { onBeforeSignup } from "@src/auth/hooks",  },}
```

src/auth/hooks.js

```
    import { HttpError } from 'wasp/server'export const onBeforeSignup = async ({  providerId,  prisma,  req,}) => {  const count = await prisma.user.count()  console.log('number of users before', count)  console.log('provider name', providerId.providerName)  console.log('provider user ID', providerId.providerUserId)  if (count > 100) {    throw new HttpError(403, 'Too many users')  }  if (providerId.providerName === 'email' && providerId.providerUserId === 'some@email.com') {    throw new HttpError(403, 'This email is not allowed')  }}
```

main.wasp

```
    app myApp {  ...  auth: {    ...    onBeforeSignup: import { onBeforeSignup } from "@src/auth/hooks",  },}
```

src/auth/hooks.ts

```
    import { HttpError } from 'wasp/server'import type { OnBeforeSignupHook } from 'wasp/server/auth'export const onBeforeSignup: OnBeforeSignupHook = async ({  providerId,  prisma,  req,}) => {  const count = await prisma.user.count()  console.log('number of users before', count)  console.log('provider name', providerId.providerName)  console.log('provider user ID', providerId.providerUserId)  if (count > 100) {    throw new HttpError(403, 'Too many users')  }  if (providerId.providerName === 'email' && providerId.providerUserId === 'some@email.com') {    throw new HttpError(403, 'This email is not allowed')  }}
```

Read more about the data the `onBeforeSignup` hook receives in the [API Reference](#the-onbeforesignup-hook).

### Executing code after the user signs up[​](#executing-code-after-the-user-signs-up "Direct link to Executing code after the user signs up")

Wasp calls the `onAfterSignup` hook after the user is created.

The `onAfterSignup` hook can be useful if you want to send the user a welcome email or perform some other action after the user signs up like syncing the user with a third-party service.

Since the `onAfterSignup` hook receives the OAuth access token, it can also be used to store the OAuth access token for the user in your database.

Works with [Email](/docs/auth/email) [Username & Password](/docs/auth/username-and-pass) [Discord](/docs/auth/social-auth/discord) [Github](/docs/auth/social-auth/github) [Google](/docs/auth/social-auth/google) [Keycloak](/docs/auth/social-auth/keycloak)

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  ...  auth: {    ...    onAfterSignup: import { onAfterSignup } from "@src/auth/hooks",  },}
```

src/auth/hooks.js

```
    export const onAfterSignup = async ({  providerId,  user,  oauth,  prisma,  req,}) => {  const count = await prisma.user.count()  console.log('number of users after', count)  console.log('user object', user)  // If this is an OAuth signup, we have the access token and uniqueRequestId  if (oauth) {    console.log('accessToken', oauth.accessToken)    console.log('uniqueRequestId', oauth.uniqueRequestId)    const id = oauth.uniqueRequestId    const data = someKindOfStore.get(id)    if (data) {      console.log('saved data for the ID', data)    }    someKindOfStore.delete(id)  }}
```

main.wasp

```
    app myApp {  ...  auth: {    ...    onAfterSignup: import { onAfterSignup } from "@src/auth/hooks",  },}
```

src/auth/hooks.ts

```
    import type { OnAfterSignupHook } from 'wasp/server/auth'export const onAfterSignup: OnAfterSignupHook = async ({  providerId,  user,  oauth,  prisma,  req,}) => {  const count = await prisma.user.count()  console.log('number of users after', count)  console.log('user object', user)  // If this is an OAuth signup, we have the access token and uniqueRequestId  if (oauth) {    console.log('accessToken', oauth.accessToken)    console.log('uniqueRequestId', oauth.uniqueRequestId)    const id = oauth.uniqueRequestId    const data = someKindOfStore.get(id)    if (data) {      console.log('saved data for the ID', data)    }    someKindOfStore.delete(id)  }}
```

Read more about the data the `onAfterSignup` hook receives in the [API Reference](#the-onaftersignup-hook).

### Executing code before the OAuth redirect[​](#executing-code-before-the-oauth-redirect "Direct link to Executing code before the OAuth redirect")

Wasp calls the `onBeforeOAuthRedirect` hook after the OAuth redirect URL is generated but before redirecting the user. This hook can access the request object sent from the client at the start of the OAuth process.

The `onBeforeOAuthRedirect` hook can be useful if you want to save some data (e.g. request query parameters) that can be used later in the OAuth flow. You can use the `uniqueRequestId` parameter to reference this data later in the `onAfterSignup` hook.

Works with [Discord](/docs/auth/social-auth/discord) [Github](/docs/auth/social-auth/github) [Google](/docs/auth/social-auth/google) [Keycloak](/docs/auth/social-auth/keycloak)

-   JavaScript
-   TypeScript

main.wasp

```
    app myApp {  ...  auth: {    ...    onBeforeOAuthRedirect: import { onBeforeOAuthRedirect } from "@src/auth/hooks",  },}
```

src/auth/hooks.js

```
    export const onBeforeOAuthRedirect = async ({  url,  uniqueRequestId,  prisma,  req,}) => {  console.log('query params before oAuth redirect', req.query)  // Saving query params for later use in the onAfterSignup hook  const id = uniqueRequestId  someKindOfStore.set(id, req.query)  return { url }}
```

main.wasp

```
    app myApp {  ...  auth: {    ...    onBeforeOAuthRedirect: import { onBeforeOAuthRedirect } from "@src/auth/hooks",  },}
```

src/auth/hooks.ts

```
    import type { OnBeforeOAuthRedirectHook } from 'wasp/server/auth'export const onBeforeOAuthRedirect: OnBeforeOAuthRedirectHook = async ({  url,  uniqueRequestId,  prisma,  req,}) => {  console.log('query params before oAuth redirect', req.query)  // Saving query params for later use in the onAfterSignup hook  const id = uniqueRequestId  someKindOfStore.set(id, req.query)  return { url }}
```

This hook's return value must be an object that looks like this: `{ url: URL }`. Wasp uses the URL to redirect the user to the OAuth provider.

Read more about the data the `onBeforeOAuthRedirect` hook receives in the [API Reference](#the-onbeforeoauthredirect-hook).

## API Reference[​](#api-reference "Direct link to API Reference")

-   JavaScript
-   TypeScript

```
    app myApp {  wasp: {    version: "^0.13.0"  },  auth: {    userEntity: User,    methods: {      ...    },    onBeforeSignup: import { onBeforeSignup } from "@src/auth/hooks",    onAfterSignup: import { onAfterSignup } from "@src/auth/hooks",    onBeforeOAuthRedirect: import { onBeforeOAuthRedirect } from "@src/auth/hooks",  },}
```

```
    app myApp {  wasp: {    version: "^0.13.0"  },  auth: {    userEntity: User,    methods: {      ...    },    onBeforeSignup: import { onBeforeSignup } from "@src/auth/hooks",    onAfterSignup: import { onAfterSignup } from "@src/auth/hooks",    onBeforeOAuthRedirect: import { onBeforeOAuthRedirect } from "@src/auth/hooks",  },}
```

### The `onBeforeSignup` hook[​](#the-onbeforesignup-hook "Direct link to the-onbeforesignup-hook")

-   JavaScript
-   TypeScript

src/auth/hooks.js

```
    import { HttpError } from 'wasp/server'export const onBeforeSignup = async ({  providerId,  prisma,  req,}) => {  // Hook code here}
```

src/auth/hooks.ts

```
    import { HttpError } from 'wasp/server'import type { OnBeforeSignupHook } from 'wasp/server/auth'export const onBeforeSignup: OnBeforeSignupHook = async ({  providerId,  prisma,  req,}) => {  // Hook code here}
```

The hook receives an object as **input** with the following properties:

-   `providerId: ProviderId`
    
    The user's provider ID is an object with two properties:
    
    -   `providerName: string`
        
        The provider's name (e.g. `'email'`, `'google'`, `'github'`)
        
    -   `providerUserId: string`
        
        The user's unique ID in the provider's system (e.g. email, Google ID, GitHub ID)
        
-   `prisma: PrismaClient`
    
    The Prisma client instance which you can use to query your database.
    
-   `req: Request`
    
    The [Express request object](https://expressjs.com/en/api.html#req) from which you can access the request headers, cookies, etc.
    

Wasp ignores this hook's **return value**.

### The `onAfterSignup` hook[​](#the-onaftersignup-hook "Direct link to the-onaftersignup-hook")

-   JavaScript
-   TypeScript

src/auth/hooks.js

```
    export const onAfterSignup = async ({  providerId,  user,  oauth,  prisma,  req,}) => {  // Hook code here}
```

src/auth/hooks.ts

```
    import type { OnAfterSignupHook } from 'wasp/server/auth'export const onAfterSignup: OnAfterSignupHook = async ({  providerId,  user,  oauth,  prisma,  req,}) => {  // Hook code here}
```

The hook receives an object as **input** with the following properties:

-   `providerId: ProviderId`
    
    The user's provider ID is an object with two properties:
    
    -   `providerName: string`
        
        The provider's name (e.g. `'email'`, `'google'`, `'github'`)
        
    -   `providerUserId: string`
        
    
    The user's unique ID in the provider's system (e.g. email, Google ID, GitHub ID)
    
-   `user: User`
    
    The user object that was created.
    
-   `oauth?: OAuthFields`
    
    This object is present only when the user is created using [Social Auth](/docs/auth/social-auth/overview). It contains the following fields:
    
    -   `accessToken: string`
        
        You can use the OAuth access token to use the provider's API on user's behalf.
        
    -   `uniqueRequestId: string`
        
        The unique request ID for the OAuth flow (you might know it as the `state` parameter in OAuth.)
        
        You can use the unique request ID to get the data saved in the `onBeforeOAuthRedirect` hook.
        
-   `prisma: PrismaClient`
    
    The Prisma client instance which you can use to query your database.
    
-   `req: Request`
    
    The [Express request object](https://expressjs.com/en/api.html#req) from which you can access the request headers, cookies, etc.
    

Wasp ignores this hook's **return value**.

### The `onBeforeOAuthRedirect` hook[​](#the-onbeforeoauthredirect-hook "Direct link to the-onbeforeoauthredirect-hook")

-   JavaScript
-   TypeScript

src/auth/hooks.js

```
    export const onBeforeOAuthRedirect = async ({  url,  uniqueRequestId,  prisma,  req,}) => {  // Hook code here  return { url }}
```

src/auth/hooks.ts

```
    import type { OnBeforeOAuthRedirectHook } from 'wasp/server/auth'export const onBeforeOAuthRedirect: OnBeforeOAuthRedirectHook = async ({  url,  uniqueRequestId,  prisma,  req,}) => {  // Hook code here  return { url }}
```

The hook receives an object as **input** with the following properties:

-   `url: URL`
    
    Wasp uses the URL for the OAuth redirect.
    
-   `uniqueRequestId: string`
    
    The unique request ID for the OAuth flow (you might know it as the `state` parameter in OAuth.)
    
    You can use the unique request ID to save data (e.g. request query params) that you can later use in the `onAfterSignup` hook.
    
-   `prisma: PrismaClient`
    
    The Prisma client instance which you can use to query your database.
    
-   `req: Request`
    
    The [Express request object](https://expressjs.com/en/api.html#req) from which you can access the request headers, cookies, etc.
    

This hook's return value must be an object that looks like this: `{ url: URL }`. Wasp uses the URL to redirect the user to the OAuth provider.

---

# Starter Templates

We created a few starter templates to help you get started with Wasp. Check out the list [below](#available-templates).

## Using a Template[​](#using-a-template "Direct link to Using a Template")

Run `wasp new` to run the interactive mode for creating a new Wasp project.

It will ask you for the project name, and then for the template to use:

```
    $ wasp newEnter the project name (e.g. my-project) ▸ MyFirstProjectChoose a starter template[1] basic (default)    Simple starter template with a single page.[2] todo-ts    Simple but well-rounded Wasp app implemented with Typescript & full-stack type safety.[3] saas    Everything a SaaS needs! Comes with Auth, ChatGPT API, Tailwind, Stripe payments and more. Check out https://opensaas.sh/ for more details.[4] embeddings    Comes with code for generating vector embeddings and performing vector similarity search.[5] ai-generated    🤖 Describe an app in a couple of sentences and have Wasp AI generate initial code for you. (experimental) ▸ 1🐝 --- Creating your project from the "basic" template... -------------------------Created new Wasp app in ./MyFirstProject directory!To run your new app, do:    cd MyFirstProject    wasp db start
```

## Available Templates[​](#available-templates "Direct link to Available Templates")

When you have a good idea for a new product, you don't want to waste your time on setting up common things like authentication, database, etc. That's why we created a few starter templates to help you get started with Wasp.

### OpenSaaS.sh template[​](#opensaassh-template "Direct link to OpenSaaS.sh template")

![SaaS Template](/assets/images/open-saas-banner-94e8d63943cb75968abc6f62b691d0c8.png)

Everything a SaaS needs! Comes with Auth, ChatGPT API, Tailwind, Stripe payments and more. Check out [https://opensaas.sh/](https://opensaas.sh/) for more details.

**Features:** Stripe Payments, OpenAI GPT API, Google Auth, SendGrid, Tailwind, & Cron Jobs

Use this template:

```
    wasp new <project-name> -t saas
```

### Vector Similarity Search Template[​](#vector-similarity-search-template "Direct link to Vector Similarity Search Template")

![Vector Similarity Search Template](/assets/images/embeddings-client-33c5c7acf7de93746c020bb041548e8a.png)

A template for generating embeddings and performing vector similarity search on your text data!

**Features:** Embeddings & vector similarity search, OpenAI Embeddings API, Vector DB (Pinecone), Tailwind, Full-stack Type Safety

Use this template:

```
    wasp new <project-name> -t embeddings
```

### Todo App w/ Typescript[​](#todo-app-w-typescript "Direct link to Todo App w/ Typescript")

A simple Todo App with Typescript and Full-stack Type Safety.

**Features:** Auth (username/password), Full-stack Type Safety

Use this template:

```
    wasp new <project-name> -t todo-ts
```

### AI Generated Starter 🤖[​](#ai-generated-starter- "Direct link to AI Generated Starter 🤖")

Using the same tech as used on [https://usemage.ai/](https://usemage.ai/), Wasp generates your custom starter template based on your project description. It will automatically generate your data model, auth, queries, actions and React pages.

*You will need to provide your own OpenAI API key to be able to use this template.*

**Features:** Generated using OpenAI's GPT models, Auth (username/password), Queries, Actions, Pages, Full-stack Type Safety

---

# Customizing the App

Each Wasp project can have only one `app` type declaration. It is used to configure your app and its components.

```
    app todoApp {  wasp: {    version: "^0.13.0"  },  title: "ToDo App",  head: [    "<link rel=\"stylesheet\" href=\"https://fonts.googleapis.com/css?family=Roboto:300,400,500&display=swap\" />"  ]}
```

We'll go through some common customizations you might want to do to your app. For more details on each of the fields, check out the [API Reference](#api-reference).

### Changing the App Title[​](#changing-the-app-title "Direct link to Changing the App Title")

You may want to change the title of your app, which appears in the browser tab, next to the favicon. You can change it by changing the `title` field of your `app` declaration:

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "BookFace"}
```

### Adding Additional Lines to the Head[​](#adding-additional-lines-to-the-head "Direct link to Adding Additional Lines to the Head")

If you are looking to add additional style sheets or scripts to your app, you can do so by adding them to the `head` field of your `app` declaration.

An example of adding extra style sheets and scripts:

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "My App",  head: [  // optional    "<link rel=\"stylesheet\" href=\"https://fonts.googleapis.com/css?family=Roboto:300,400,500&display=swap\" />",    "<script src=\"https://cdnjs.cloudflare.com/ajax/libs/Chart.js/2.9.3/Chart.min.js\"></script>",    "<meta name=\"viewport\" content=\"minimum-scale=1, initial-scale=1, width=device-width\" />"  ]}
```

## API Reference[​](#api-reference "Direct link to API Reference")

```
    app todoApp {  wasp: {    version: "^0.13.0"  },  title: "ToDo App",  head: [    "<link rel=\"stylesheet\" href=\"https://fonts.googleapis.com/css?family=Roboto:300,400,500&display=swap\" />"  ],  auth: {    // ...  },  client: {    // ...  },  server: {    // ...  },  db: {    // ...  },  emailSender: {    // ...  },  webSocket: {    // ...  }}
```

The `app` declaration has the following fields:

-   `wasp: dict` required Wasp compiler configuration. It is a dictionary with a single field:
    
    -   `version: string` required
        
        The version specifies which versions of Wasp are compatible with the app. It should contain a valid [SemVer range](https://github.com/npm/node-semver#ranges)
        
        info
        
        For now, the version field only supports caret ranges (i.e., `^x.y.z`). Support for the full specification will come in a future version of Wasp
        
-   `title: string` required
    
    Title of your app. It will appear in the browser tab, next to the favicon.
    
-   `head: [string]`
    
    List of additional lines (e.g. `<link>` or `<script>` tags) to be included in the `<head>` of your HTML document.
    

The rest of the fields are covered in dedicated sections of the docs:

-   `auth: dict`
    
    Authentication configuration. Read more in the [authentication section](/docs/auth/overview) of the docs.
    
-   `client: dict`
    
    Configuration for the client side of your app. Read more in the [client configuration section](/docs/project/client-config) of the docs.
    
-   `server: dict`
    
    Configuration for the server side of your app. Read more in the [server configuration section](/docs/project/server-config) of the docs.
    
-   `db: dict`
    
    Database configuration. Read more in the [database configuration section](/docs/data-model/backends) of the docs.
    
-   `emailSender: dict`
    
    Email sender configuration. Read more in the [email sending section](/docs/advanced/email) of the docs.
    
-   `webSocket: dict`
    
    WebSocket configuration. Read more in the [WebSocket section](/docs/advanced/web-sockets) of the docs.

---

# Client Config

You can configure the client using the `client` field inside the `app` declaration:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.jsx",    setupFn: import mySetupFunction from "@src/myClientSetupCode.js"  }}
```

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.tsx",    setupFn: import mySetupFunction from "@src/myClientSetupCode.ts"  }}
```

## Root Component[​](#root-component "Direct link to Root Component")

Wasp gives you the option to define a "wrapper" component for your React app.

It can be used for a variety of purposes, but the most common ones are:

-   Defining a common layout for your application.
-   Setting up various providers that your application needs.

### Defining a Common Layout[​](#defining-a-common-layout "Direct link to Defining a Common Layout")

Let's define a common layout for your application:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.jsx",  }}
```

src/Root.jsx

```
    export default function Root({ children }) {  return (    <div>      <header>        <h1>My App</h1>      </header>      {children}      <footer>        <p>My App footer</p>      </footer>    </div>  )}
```

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.tsx",  }}
```

src/Root.tsx

```
    export default function Root({ children }: { children: React.ReactNode }) {  return (    <div>      <header>        <h1>My App</h1>      </header>      {children}      <footer>        <p>My App footer</p>      </footer>    </div>  )}
```

### Setting up a Provider[​](#setting-up-a-provider "Direct link to Setting up a Provider")

This is how to set up various providers that your application needs:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.jsx",  }}
```

src/Root.jsx

```
    import store from './store'import { Provider } from 'react-redux'export default function Root({ children }) {  return <Provider store={store}>{children}</Provider>}
```

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.tsx",  }}
```

src/Root.tsx

```
    import store from './store'import { Provider } from 'react-redux'export default function Root({ children }: { children: React.ReactNode }) {  return <Provider store={store}>{children}</Provider>}
```

As long as you render the children, you can do whatever you want in your root component.

Read more about the root component in the [API Reference](#rootcomponent-extimport).

## Setup Function[​](#setup-function "Direct link to Setup Function")

`setupFn` declares a function that Wasp executes on the client before everything else.

### Running Some Code[​](#running-some-code "Direct link to Running Some Code")

We can run any code we want in the setup function.

For example, here's a setup function that logs a message every hour:

-   JavaScript
-   TypeScript

src/myClientSetupCode.js

```
    export default async function mySetupFunction() {  let count = 1  setInterval(    () => console.log(`You have been online for ${count++} hours.`),    1000 * 60 * 60  )}
```

src/myClientSetupCode.ts

```
    export default async function mySetupFunction(): Promise<void> {  let count = 1  setInterval(    () => console.log(`You have been online for ${count++} hours.`),    1000 * 60 * 60  )}
```

### Overriding Default Behaviour for Queries[​](#overriding-default-behaviour-for-queries "Direct link to Overriding Default Behaviour for Queries")

info

You can change the options for a **single** Query using the `options` object, as described [here](/docs/data-model/operations/queries#the-usequery-hook-1).

Wasp's `useQuery` hook uses `react-query`'s `useQuery` hook under the hood. Since `react-query` comes configured with aggressive but sane default options, you most likely won't have to change those defaults for all Queries.

If you do need to change the global defaults, you can do so inside the client setup function.

Wasp exposes a `configureQueryClient` hook that lets you configure *react-query*'s `QueryClient` object:

-   JavaScript
-   TypeScript

src/myClientSetupCode.js

```
    import { configureQueryClient } from 'wasp/client/operations'export default async function mySetupFunction() {  // ... some setup  configureQueryClient({    defaultOptions: {      queries: {        staleTime: Infinity,      },    },  })  // ... some more setup}
```

src/myClientSetupCode.ts

```
    import { configureQueryClient } from 'wasp/client/operations'export default async function mySetupFunction(): Promise<void> {  // ... some setup  configureQueryClient({    defaultOptions: {      queries: {        staleTime: Infinity,      },    },  })  // ... some more setup}
```

Make sure to pass in an object expected by the `QueryClient`'s constructor, as explained in [react-query's docs](https://tanstack.com/query/v4/docs/reference/QueryClient).

Read more about the setup function in the [API Reference](#setupfn-extimport).

## Base Directory[​](#base-directory "Direct link to Base Directory")

If you need to serve the client from a subdirectory, you can use the `baseDir` option:

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    baseDir: "/my-app",  }}
```

This means that if you serve your app from `https://example.com/my-app`, the router will work correctly, and all the assets will be served from `https://example.com/my-app`.

Setting the correct env variable

If you set the `baseDir` option, make sure that the `WASP_WEB_CLIENT_URL` env variable also includes that base directory.

For example, if you are serving your app from `https://example.com/my-app`, the `WASP_WEB_CLIENT_URL` should be also set to `https://example.com/my-app`, and not just `https://example.com`.

## API Reference[​](#api-reference "Direct link to API Reference")

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.jsx",    setupFn: import mySetupFunction from "@src/myClientSetupCode.js"  }}
```

main.wasp

```
    app MyApp {  title: "My app",  // ...  client: {    rootComponent: import Root from "@src/Root.tsx",    setupFn: import mySetupFunction from "@src/myClientSetupCode.ts",    baseDir: "/my-app",  }}
```

Client has the following options:

-   #### `rootComponent: ExtImport`[​](#rootcomponent-extimport "Direct link to rootcomponent-extimport")
    
    `rootComponent` defines the root component of your client application. It is expected to be a React component, and Wasp will use it to wrap your entire app. It must render its children, which are the actual pages of your application.
    
    Here's an example of a root component that both sets up a provider and renders a custom layout:
    
    -   JavaScript
    -   TypeScript
    
    src/Root.jsx
    
    ```
        import store from './store'import { Provider } from 'react-redux'export default function Root({ children }) {  return (    <Provider store={store}>      <Layout>{children}</Layout>    </Provider>  )}function Layout({ children }) {  return (    <div>      <header>        <h1>My App</h1>      </header>      {children}      <footer>        <p>My App footer</p>      </footer>    </div>  )}
    ```
    
    src/Root.tsx
    
    ```
        import store from './store'import { Provider } from 'react-redux'export default function Root({ children }: { children: React.ReactNode }) {  return (    <Provider store={store}>      <Layout>{children}</Layout>    </Provider>  )}function Layout({ children }: { children: React.ReactNode }) {  return (    <div>      <header>        <h1>My App</h1>      </header>      {children}      <footer>        <p>My App footer</p>      </footer>    </div>  )}
    ```
    
-   #### `setupFn: ExtImport`[​](#setupfn-extimport "Direct link to setupfn-extimport")
    
    You can use this function to perform any custom setup (e.g., setting up client-side periodic jobs).
    
    -   JavaScript
    -   TypeScript
    
    src/myClientSetupCode.js
    
    ```
        export default async function mySetupFunction() {  // Run some code}
    ```
    
    src/myClientSetupCode.ts
    
    ```
        export default async function mySetupFunction(): Promise<void> {  // Run some code}
    ```
    
-   #### `baseDir: String`[​](#basedir-string "Direct link to basedir-string")
    
    If you need to serve the client from a subdirectory, you can use the `baseDir` option.
    
    If you set `baseDir` to `/my-app` for example, that will make Wasp set the `basename` prop of the `Router` to `/my-app`. It will also set the `base` option of the Vite config to `/my-app`.
    
    This means that if you serve your app from `https://example.com/my-app`, the router will work correctly, and all the assets will be served from `https://example.com/my-app`.
    
    Setting the correct env variable
    
    If you set the `baseDir` option, make sure that the `WASP_WEB_CLIENT_URL` env variable also includes that base directory.
    
    For example, if you are serving your app from `https://example.com/my-app`, the `WASP_WEB_CLIENT_URL` should be also set to `https://example.com/my-app`, and not just `https://example.com`.

---

# Server Config

You can configure the behavior of the server via the `server` field of `app` declaration:

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  // ...  server: {    setupFn: import { mySetupFunction } from "@src/myServerSetupCode.js",    middlewareConfigFn: import { myMiddlewareConfigFn } from "@src/myServerSetupCode.js"  }}
```

main.wasp

```
    app MyApp {  title: "My app",  // ...  server: {    setupFn: import { mySetupFunction } from "@src/myServerSetupCode.js",    middlewareConfigFn: import { myMiddlewareConfigFn } from "@src/myServerSetupCode.js"  }}
```

## Setup Function[​](#setup-function "Direct link to Setup Function")

### Adding a Custom Route[​](#adding-a-custom-route "Direct link to Adding a Custom Route")

As an example, adding a custom route would look something like:

-   JavaScript
-   TypeScript

src/myServerSetupCode.ts

```
    export const mySetupFunction = async ({ app }) => {  addCustomRoute(app)}function addCustomRoute(app) {  app.get('/customRoute', (_req, res) => {    res.send('I am a custom route')  })}
```

src/myServerSetupCode.ts

```
    import { ServerSetupFn } from 'wasp/server'import { Application } from 'express'export const mySetupFunction: ServerSetupFn = async ({ app }) => {  addCustomRoute(app)}function addCustomRoute(app: Application) {  app.get('/customRoute', (_req, res) => {    res.send('I am a custom route')  })}
```

### Storing Some Values for Later Use[​](#storing-some-values-for-later-use "Direct link to Storing Some Values for Later Use")

In case you want to store some values for later use, or to be accessed by the [Operations](/docs/data-model/operations/overview) you do that in the `setupFn` function.

Dummy example of such function and its usage:

-   JavaScript
-   TypeScript

src/myServerSetupCode.js

```
    let someResource = undefinedexport const mySetupFunction = async () => {  // Let's pretend functions setUpSomeResource and startSomeCronJob  // are implemented below or imported from another file.  someResource = await setUpSomeResource()  startSomeCronJob()}export const getSomeResource = () => someResource
```

src/queries.js

```
    import { getSomeResource } from './myServerSetupCode.js'...export const someQuery = async (args, context) => {  const someResource = getSomeResource()  return queryDataFromSomeResource(args, someResource)}
```

src/myServerSetupCode.ts

```
    import { type ServerSetupFn } from 'wasp/server'let someResource = undefinedexport const mySetupFunction: ServerSetupFn = async () => {  // Let's pretend functions setUpSomeResource and startSomeCronJob  // are implemented below or imported from another file.  someResource = await setUpSomeResource()  startSomeCronJob()  }export const getSomeResource = () => someResource
```

src/queries.ts

```
    import { type SomeQuery } from 'wasp/server/operations'import { getSomeResource } from './myServerSetupCode.js'...export const someQuery: SomeQuery<...> = async (args, context) => {  const someResource = getSomeResource()  return queryDataFromSomeResource(args, someResource)}
```

note

The recommended way is to put the variable in the same module where you defined the setup function and then expose additional functions for reading those values, which you can then import directly from Operations and use.

This effectively turns your module into a singleton whose construction is performed on server start.

Read more about [server setup function](#setupfn-extimport) below.

## Middleware Config Function[​](#middleware-config-function "Direct link to Middleware Config Function")

You can configure the global middleware via the `middlewareConfigFn`. This will modify the middleware stack for all operations and APIs.

Read more about [middleware config function](#middlewareconfigfn-extimport) below.

## API Reference[​](#api-reference "Direct link to API Reference")

-   JavaScript
-   TypeScript

main.wasp

```
    app MyApp {  title: "My app",  // ...  server: {    setupFn: import { mySetupFunction } from "@src/myServerSetupCode.js",    middlewareConfigFn: import { myMiddlewareConfigFn } from "@src/myServerSetupCode.js"  }}
```

main.wasp

```
    app MyApp {  title: "My app",  // ...  server: {    setupFn: import { mySetupFunction } from "@src/myServerSetupCode.js",    middlewareConfigFn: import { myMiddlewareConfigFn } from "@src/myServerSetupCode.js"  }}
```

`app.server` is a dictionary with the following fields:

-   #### `setupFn: ExtImport`[​](#setupfn-extimport "Direct link to setupfn-extimport")
    
    `setupFn` declares a function that will be executed on server start. This function is expected to be async and will be awaited before the server starts accepting any requests.
    
    It allows you to do any custom setup, e.g. setting up additional database/websockets or starting cron/scheduled jobs.
    
    The `setupFn` function receives the `express.Application` and the `http.Server` instances as part of its context. They can be useful for setting up any custom server logic.
    
    -   JavaScript
    -   TypeScript
    
    src/myServerSetupCode.js
    
    ```
        export const mySetupFunction = async () => {  await setUpSomeResource()}
    ```
    
    Types for the setup function and its context are as follows:
    
    wasp/server
    
    ```
        export type ServerSetupFn = (context: ServerSetupFnContext) => Promise<void>export type ServerSetupFnContext = {  app: Application // === express.Application  server: Server // === http.Server}
    ```
    
    src/myServerSetupCode.ts
    
    ```
        import { type ServerSetupFn } from 'wasp/server'export const mySetupFunction: ServerSetupFn = async () => {  await setUpSomeResource()}
    ```
    
-   #### `middlewareConfigFn: ExtImport`[​](#middlewareconfigfn-extimport "Direct link to middlewareconfigfn-extimport")
    
    The import statement to an Express middleware config function. This is a global modification affecting all operations and APIs. See more in the [configuring middleware section](/docs/advanced/middleware-config#1-customize-global-middleware).

---

# Static Asset Handling

## Importing an Asset as URL[​](#importing-an-asset-as-url "Direct link to Importing an Asset as URL")

Importing a static asset (e.g. an image) will return its URL. For example:

-   JavaScript
-   TypeScript

src/App.jsx

```
    import imgUrl from './img.png'function App() {  return <img src={imgUrl} alt="img" />}
```

src/App.tsx

```
    import imgUrl from './img.png'function App() {  return <img src={imgUrl} alt="img" />}
```

For example, `imgUrl` will be `/img.png` during development, and become `/assets/img.2d8efhg.png` in the production build.

This is what you want to use most of the time, as it ensures that the asset file exists and is included in the bundle.

We are using Vite under the hood, read more about importing static assets in Vite's [docs](https://vitejs.dev/guide/assets.html#importing-asset-as-url).

## The `public` Directory[​](#the-public-directory "Direct link to the-public-directory")

If you have assets that are:

-   Never referenced in source code (e.g. robots.txt)
-   Must retain the exact same file name (without hashing)
-   ...or you simply don't want to have to import an asset first just to get its URL

Then you can place the asset in the `public` directory at the root of your project:

```
    .└── public    ├── favicon.ico    └── robots.txt
```

Assets in this directory will be served at root path `/` during development and copied to the root of the dist directory as-is.

For example, if you have a file `favicon.ico` in the `public` directory, and your app is hosted at `https://myapp.com`, it will be made available at `https://myapp.com/favicon.ico`.

Usage in client code

Note that:

-   You should always reference public assets using root absolute path
    -   for example, `public/icon.png` should be referenced in source code as `/icon.png`.
-   Assets in the `public` directory **cannot be imported** from .

---

# Env Variables

**Environment variables** are used to configure projects based on the context in which they run. This allows them to exhibit different behaviors in different environments, such as development, staging, or production.

For instance, *during development*, you may want your project to connect to a local development database running on your machine, but *in production*, you may prefer it to connect to the production database. Similarly, in development, you may want to use a test Stripe account, while in production, your app should use a real Stripe account.

While some env vars are required by Wasp, such as the database connection or secrets for social auth, you can also define your env vars for any other useful purposes, and then access them in the code.

In Wasp, you can use environment variables in both the client and the server code.

## Client Env Vars[​](#client-env-vars "Direct link to Client Env Vars")

Client environment variables are embedded into the client code during the build and shipping process, making them public and readable by anyone. Therefore, you should **never store secrets in them** (such as secret API keys -> you can provide those to server instead).

To enable Wasp to pick them up, client env vars must be prefixed with `REACT_APP_`, for example: `REACT_APP_SOME_VAR_NAME=...`.

You can read them from the client code like this:

-   JavaScript
-   TypeScript

src/App.js

```
    console.log(import.meta.env.REACT_APP_SOME_VAR_NAME)
```

src/App.ts

```
    console.log(import.meta.env.REACT_APP_SOME_VAR_NAME)
```

Check below on how to define them.

## Server Env Vars[​](#server-env-vars "Direct link to Server Env Vars")

In server environment variables, you can store secret values (e.g. secret API keys) since they are not publicly readable. You can define them without any special prefix, such as `SOME_VAR_NAME=...`.

You can read the env vars from server code like this:

-   JavaScript
-   TypeScript

```
    console.log(process.env.SOME_VAR_NAME)
```

```
    console.log(process.env.SOME_VAR_NAME)
```

Check below on how to define them.

## Defining Env Vars in Development[​](#defining-env-vars-in-development "Direct link to Defining Env Vars in Development")

During development (`wasp start`), there are two ways to provide env vars to your Wasp project:

1.  Using `.env` files. **(recommended)**
2.  Using shell. (useful for overrides)

### 1\. Using .env (dotenv) Files[​](#1-using-env-dotenv-files "Direct link to 1. Using .env (dotenv) Files")

![Env vars usage in development](/assets/images/prod_dev_fade-e4097e7d9b64c62ca95bfde692e5115d.svg)

This is the recommended method for providing env vars to your Wasp project during development.

In the root of your Wasp project you can create two distinct files:

-   `.env.server` for env vars that will be provided to the server.
    
    Variables are defined in these files in the form of `NAME=VALUE`, for example:
    
    .env.server
    
    ```
        DATABASE_URL=postgresql://localhost:5432SOME_VAR_NAME=somevalue
    ```
    
-   `.env.client` for env vars that will be provided to the client.
    
    Variables are defined in these files in the form of `NAME=VALUE`, for example:
    
    .env.client
    
    ```
        REACT_APP_SOME_VAR_NAME=somevalue
    ```
    

`.env.server` should not be committed to version control as it can contain secrets, while `.env.client` can be versioned as it must not contain any secrets. By default, in the `.gitignore` file that comes with a new Wasp app, we ignore all dotenv files.

Dotenv files

`dotenv` files are a popular method for storing configuration: to learn more about them in general, check out the [dotenv npm package](https://www.npmjs.com/package/dotenv).

### 2\. Using Shell[​](#2-using-shell "Direct link to 2. Using Shell")

If you set environment variables in the shell where you run your Wasp commands (e.g., `wasp start`), Wasp will recognize them.

You can set environment variables in the `.profile` or a similar file, which will set them permanently, or you can set them temporarily by defining them at the start of a command (`SOME_VAR_NAME=SOMEVALUE wasp start`).

This is not specific to Wasp and is simply how environment variables can be set in the shell.

Defining environment variables in this way can be cumbersome even for a single project and even more challenging to manage if you have multiple Wasp projects. Therefore, we do not recommend this as a default method for providing environment variables to Wasp projects during development, you should use .env files instead. However, it can be useful for occasionally **overriding** specific environment variables because environment variables set this way **take precedence over those defined in `.env` files**.

## Defining Env Vars in Production[​](#defining-env-vars-in-production "Direct link to Defining Env Vars in Production")

While in development, we had the option of using `.env.client` and `.env.server` files which made it easy to define and manage env vars. However, for production, `.env.client` and `.env.server` files will be ignored, and we need to provide env vars differently.

![Env vars usage in development and production](/assets/images/prod_dev_fade_2-d0ff1e438a29011a68bcf630a9470254.svg)

### Client Env Vars[​](#client-env-vars-1 "Direct link to Client Env Vars")

Client env vars are embedded into the client code during the build process, making them public and readable by anyone. Therefore, you should **never store secrets in them** (such as secret API keys).

When building for production `.env.client` will be ignored, since it is meant to be used only during development. Instead, you should provide the production client env vars directly to the build command that turns client code into static files:

```
    REACT_APP_SOME_VAR_NAME=somevalue REACT_APP_SOME_OTHER_VAR_NAME=someothervalue npm run build
```

Check the [deployment docs](/docs/advanced/deployment/manually#3-deploying-the-web-client-frontend) for more details.

Also, notice that you can't and shouldn't provide env vars to the client code by setting them on the hosting provider where you deployed them (unlike server env vars, where this is how you should do it). Your client code will ignore those, as at that point client code is just static files.

How it works

What happens behind the scenes is that Wasp will replace all occurrences of `import.meta.env.REACT_APP_SOME_VAR_NAME` in your client code with the env var value you provided. This is done during the build process, so the value is embedded into the static files produced from the client code.

Read more about it in Vite's [docs](https://vitejs.dev/guide/env-and-mode.html#production-replacement).

### Server Env Vars[​](#server-env-vars-1 "Direct link to Server Env Vars")

When building for production `.env.server` will be ignored, since it is meant to be used only during development.

You can provide production env vars to your server code in production by defining them and making them available on the server where your server code is running.

Setting this up will highly depend on where you are deploying your Wasp project, but in general it comes down to defining the env vars via mechanisms that your hosting provider provides.

For example, if you deploy your project to [Fly](https://fly.io), you can define them using the `flyctl` CLI tool:

```
    flyctl secrets set SOME_VAR_NAME=somevalue
```

You can read a lot more details in the [deployment section](/docs/advanced/deployment/manually) of the docs. We go into detail on how to define env vars for each deployment option.

---

# Testing

info

Wasp is in beta, so keep in mind there might be some kinks / bugs, and possibly some changes with testing support in the future. If you encounter any issues, reach out to us on [Discord](https://discord.gg/rzdnErX) and we will make sure to help you out!

## Testing Your React App[​](#testing-your-react-app "Direct link to Testing Your React App")

Wasp enables you to quickly and easily write both unit tests and React component tests for your frontend code. Because Wasp uses [Vite](https://vitejs.dev/), we support testing web apps through [Vitest](https://vitest.dev/).

Included Libraries

[`vitest`](https://www.npmjs.com/package/vitest): Unit test framework with native Vite support.

[`@vitest/ui`](https://www.npmjs.com/package/@vitest/ui): A nice UI for seeing your test results.

[`jsdom`](https://www.npmjs.com/package/jsdom): A web browser test environment for Node.js.

[`@testing-library/react`](https://www.npmjs.com/package/@testing-library/react) / [`@testing-library/jest-dom`](https://www.npmjs.com/package/@testing-library/jest-dom): Testing helpers.

[`msw`](https://www.npmjs.com/package/msw): A server mocking library.

### Writing Tests[​](#writing-tests "Direct link to Writing Tests")

For Wasp to pick up your tests, they should be placed within the `src` directory and use an extension that matches [these glob patterns](https://vitest.dev/config#include). Some of the file names that Wasp will pick up as tests:

-   `yourFile.test.ts`
-   `YourComponent.spec.jsx`

Within test files, you can import your other source files as usual. For example, if you have a component `Counter.jsx`, you test it by creating a file in the same directory called `Counter.test.jsx` and import the component with `import Counter from './Counter'`.

### Running Tests[​](#running-tests "Direct link to Running Tests")

Running `wasp test client` will start Vitest in watch mode and recompile your Wasp project when changes are made.

-   If you want to see a real-time UI, pass `--ui` as an option.
-   To run the tests just once, use `wasp test client run`.

All arguments after `wasp test client` are passed directly to the Vitest CLI, so check out [their documentation](https://vitest.dev/guide/cli.html) for all of the options.

Be Careful

You should not run `wasp test` while `wasp start` is running. Both will try to compile your project to `.wasp/out`.

### React Testing Helpers[​](#react-testing-helpers "Direct link to React Testing Helpers")

Wasp provides several functions to help you write React tests:

-   `renderInContext`: Takes a React component, wraps it inside a `QueryClientProvider` and `Router`, and renders it. This is the function you should use to render components in your React component tests.
    
    ```
        import { renderInContext } from "wasp/client/test";renderInContext(<MainPage />);
    ```
    
-   `mockServer`: Sets up the mock server and returns an object containing the `mockQuery` and `mockApi` utilities. This should be called outside of any test case, in each file that wants to use those helpers.
    
    ```
        import { mockServer } from "wasp/client/test";const { mockQuery, mockApi } = mockServer();
    ```
    
    -   `mockQuery`: Takes a Wasp [query](/docs/data-model/operations/queries) to mock and the JSON data it should return.
        
        ```
            import { getTasks } from "wasp/client/operations";mockQuery(getTasks, []);
        ```
        
        -   Helpful when your component uses `useQuery`.
        -   Behind the scenes, Wasp uses [`msw`](https://npmjs.com/package/msw) to create a server request handle that responds with the specified data.
        -   Mock are cleared between each test.
    -   `mockApi`: Similar to `mockQuery`, but for [APIs](/docs/advanced/apis). Instead of a Wasp query, it takes a route containing an HTTP method and a path.
        
        ```
            import { HttpMethod } from "wasp/client";mockApi({ method: HttpMethod.Get, path: "/foor/bar" }, { res: "hello" });
        ```
        

## Testing Your Server-Side Code[​](#testing-your-server-side-code "Direct link to Testing Your Server-Side Code")

Wasp currently does not provide a way to test your server-side code, but we will be adding support soon. You can track the progress at [this GitHub issue](https://github.com/wasp-lang/wasp/issues/110) and express your interest by commenting.

## Examples[​](#examples "Direct link to Examples")

You can see some tests in a Wasp project [here](https://github.com/wasp-lang/wasp/blob/release/waspc/examples/todoApp/src/pages/auth/helpers.test.ts).

### Client Unit Tests[​](#client-unit-tests "Direct link to Client Unit Tests")

-   JavaScript
-   TypeScript

src/helpers.js

```
    export function areThereAnyTasks(tasks) {  return tasks.length === 0;}
```

src/helpers.test.js

```
    import { test, expect } from "vitest";import { areThereAnyTasks } from "./helpers";test("areThereAnyTasks", () => {  expect(areThereAnyTasks([])).toBe(false);});
```

src/helpers.ts

```
    import { type Task } from "wasp/entities";export function areThereAnyTasks(tasks: Task[]): boolean {  return tasks.length === 0;}
```

src/helpers.test.ts

```
    import { test, expect } from "vitest";import { areThereAnyTasks } from "./helpers";test("areThereAnyTasks", () => {  expect(areThereAnyTasks([])).toBe(false);});
```

### React Component Tests[​](#react-component-tests "Direct link to React Component Tests")

-   JavaScript
-   TypeScript

src/Todo.jsx

```
    import { useQuery, getTasks } from "wasp/client/operations";const Todo = (_props) => {  const { data: tasks } = useQuery(getTasks);  return (    <ul>      {tasks &&        tasks.map((task) => (          <li key={task.id}>            <input type="checkbox" value={task.isDone} />            {task.description}          </li>        ))}    </ul>  );};
```

src/Todo.test.jsx

```
    import { test, expect } from "vitest";import { screen } from "@testing-library/react";import { mockServer, renderInContext } from "wasp/client/test";import { getTasks } from "wasp/client/operations";import Todo from "./Todo";const { mockQuery } = mockServer();const mockTasks = [  {    id: 1,    description: "test todo 1",    isDone: true,    userId: 1,  },];test("handles mock data", async () => {  mockQuery(getTasks, mockTasks);  renderInContext(<Todo />);  await screen.findByText("test todo 1");  expect(screen.getByRole("checkbox")).toBeChecked();  screen.debug();});
```

src/Todo.tsx

```
    import { useQuery, getTasks } from "wasp/client/operations";const Todo = (_props: {}) => {  const { data: tasks } = useQuery(getTasks);  return (    <ul>      {tasks &&        tasks.map((task) => (          <li key={task.id}>            <input type="checkbox" value={task.isDone} />            {task.description}          </li>        ))}    </ul>  );};
```

src/Todo.test.tsx

```
    import { test, expect } from "vitest";import { screen } from "@testing-library/react";import { mockServer, renderInContext } from "wasp/client/test";import { getTasks } from "wasp/client/operations";import Todo from "./Todo";const { mockQuery } = mockServer();const mockTasks = [  {    id: 1,    description: "test todo 1",    isDone: true,    userId: 1,  },];test("handles mock data", async () => {  mockQuery(getTasks, mockTasks);  renderInContext(<Todo />);  await screen.findByText("test todo 1");  expect(screen.getByRole("checkbox")).toBeChecked();  screen.debug();});
```

### Testing With Mocked APIs[​](#testing-with-mocked-apis "Direct link to Testing With Mocked APIs")

-   JavaScript
-   TypeScript

src/Todo.jsx

```
    import { api } from "wasp/client/api";const Todo = (_props) => {  const [tasks, setTasks] = useState([]);  useEffect(() => {    api      .get("/tasks")      .then((res) => res.json())      .then((tasks) => setTasks(tasks))      .catch((err) => window.alert(err));  });  return (    <ul>      {tasks &&        tasks.map((task) => (          <li key={task.id}>            <input type="checkbox" value={task.isDone} />            {task.description}          </li>        ))}    </ul>  );};
```

src/Todo.test.jsx

```
    import { test, expect } from "vitest";import { screen } from "@testing-library/react";import { mockServer, renderInContext } from "wasp/client/test";import Todo from "./Todo";const { mockApi } = mockServer();const mockTasks = [  {    id: 1,    description: "test todo 1",    isDone: true,    userId: 1,  },];test("handles mock data", async () => {  mockApi("/tasks", { res: mockTasks });  renderInContext(<Todo />);  await screen.findByText("test todo 1");  expect(screen.getByRole("checkbox")).toBeChecked();  screen.debug();});
```

src/Todo.tsx

```
    import { type Task } from "wasp/entities";import { api } from "wasp/client/api";const Todo = (_props: {}) => {  const [tasks, setTasks] = useState<Task>([]);  useEffect(() => {    api      .get("/tasks")      .then((res) => res.json() as Task[])      .then((tasks) => setTasks(tasks))      .catch((err) => window.alert(err));  });  return (    <ul>      {tasks &&        tasks.map((task) => (          <li key={task.id}>            <input type="checkbox" value={task.isDone} />            {task.description}          </li>        ))}    </ul>  );};
```

src/Todo.test.tsx

```
    import { test, expect } from "vitest";import { screen } from "@testing-library/react";import { mockServer, renderInContext } from "wasp/client/test";import Todo from "./Todo";const { mockApi } = mockServer();const mockTasks = [  {    id: 1,    description: "test todo 1",    isDone: true,    userId: 1,  },];test("handles mock data", async () => {  mockApi("/tasks", mockTasks);  renderInContext(<Todo />);  await screen.findByText("test todo 1");  expect(screen.getByRole("checkbox")).toBeChecked();  screen.debug();});
```

---

# Dependencies

In a Wasp project, dependencies are defined in a standard way for JavaScript projects: using the [package.json](https://docs.npmjs.com/cli/configuring-npm/package-json) file, located at the root of your project. You can list your dependencies under the `dependencies` or `devDependencies` fields.

### Adding a New Dependency[​](#adding-a-new-dependency "Direct link to Adding a New Dependency")

To add a new package, like `date-fns` (a great date handling library), you use `npm`:

```
    npm install date-fns
```

This command will add the package in the `dependencies` section of your `package.json` file.

You will notice that there are some other packages in the `dependencies` section, like `react` and `wasp`. These are the packages that Wasp uses internally, and you should not modify or remove them.

### Using Packages that are Already Used by Wasp Internally[​](#using-packages-that-are-already-used-by-wasp-internally "Direct link to Using Packages that are Already Used by Wasp Internally")

In the current version of Wasp, if Wasp is already internally using a certain dependency (e.g. React) with a certain version specified, you are not allowed to define that same npm dependency yourself while specifying *a different version*.

If you do that, you will get an error message telling you which exact version you have to use for that dependency. This means Wasp *dictates exact versions of certain packages*, so for example you can't choose the version of React you want to use.

note

We are currently working on a restructuring that will solve this and some other quirks: check [issue #734](https://github.com/wasp-lang/wasp/issues/734) to follow our progress.

---

# CSS Frameworks

## Tailwind[​](#tailwind "Direct link to Tailwind")

To enable support for Tailwind in your project, you need to add two config files — [`tailwind.config.cjs`](https://tailwindcss.com/docs/configuration#configuration-options) and `postcss.config.cjs` — to the root directory.

With these files present, Wasp installs the necessary dependencies and copies your configuration to the generated project. You can then use [Tailwind CSS directives](https://tailwindcss.com/docs/functions-and-directives#directives) in your CSS and Tailwind classes on your React components.

tree .

```
    .├── main.wasp├── package.json├── src│   ├── Main.css│   ├── MainPage.jsx│   ├── vite-env.d.ts│   └── waspLogo.png├── public├── tsconfig.json├── vite.config.ts├── postcss.config.cjs└── tailwind.config.cjs
```

Tailwind not working?

If you can not use Tailwind after adding the required config files, make sure to restart `wasp start`. This is sometimes needed to ensure that Wasp picks up the changes and enables Tailwind integration.

### Enabling Tailwind Step-by-Step[​](#enabling-tailwind-step-by-step "Direct link to Enabling Tailwind Step-by-Step")

caution

Make sure to use the `.cjs` extension for these config files, if you name them with a `.js` extension, Wasp will not detect them.

1.  Add `./tailwind.config.cjs`.
    
    ./tailwind.config.cjs
    
    ```
        const { resolveProjectPath } = require('wasp/dev')/** @type {import('tailwindcss').Config} */module.exports = {  content: [resolveProjectPath('./src/**/*.{js,jsx,ts,tsx}')],  theme: {    extend: {},  },  plugins: [],}
    ```
    
2.  Add `./postcss.config.cjs`.
    
    ./postcss.config.cjs
    
    ```
        module.exports = {  plugins: {    tailwindcss: {},    autoprefixer: {},  },}
    ```
    
3.  Import Tailwind into your CSS file. For example, in a new project you might import Tailwind into `Main.css`.
    
    ./src/Main.css
    
    ```
        @tailwind base;@tailwind components;@tailwind utilities;/* ... */
    ```
    
4.  Start using Tailwind 🥳
    
    ./src/MainPage.jsx
    
    ```
        // ...<h1 className="text-3xl font-bold underline">  Hello world!</h1>// ...
    ```
    

### Adding Tailwind Plugins[​](#adding-tailwind-plugins "Direct link to Adding Tailwind Plugins")

To add Tailwind plugins, install them as npm development [dependencies](/docs/project/dependencies) and add them to the plugins list in your `tailwind.config.cjs` file:

```
    npm install -D @tailwindcss/formsnpm install -D @tailwindcss/typography
```

and also

./tailwind.config.cjs

```
    /** @type {import('tailwindcss').Config} */module.exports = {  // ...  plugins: [    require('@tailwindcss/forms'),    require('@tailwindcss/typography'),  ],  // ...}
```

---

# Custom Vite Config

Wasp uses [Vite](https://vitejs.dev/) to serve the client during development and bundling it for production. If you want to customize the Vite config, you can do that by editing the `vite.config.ts` file in your project root directory.

Wasp will use your config and **merge** it with the default Wasp's Vite config.

Vite config customization can be useful for things like:

-   Adding custom Vite plugins.
-   Customising the dev server.
-   Customising the build process.

Be careful with making changes to the Vite config, as it can break the Wasp's client build process. Check out the default Vite config [here](https://github.com/wasp-lang/wasp/blob/release/waspc/data/Generator/templates/react-app/vite.config.ts) to see what you can change.

## Examples[​](#examples "Direct link to Examples")

Below are some examples of how you can customize the Vite config.

### Changing the Dev Server Behaviour[​](#changing-the-dev-server-behaviour "Direct link to Changing the Dev Server Behaviour")

If you want to stop Vite from opening the browser automatically when you run `wasp start`, you can do that by customizing the `open` option.

-   JavaScript
-   TypeScript

vite.config.js

```
    export default {  server: {    open: false,  },}
```

vite.config.ts

```
    import { defineConfig } from 'vite'export default defineConfig({  server: {    open: false,  },})
```

### Custom Dev Server Port[​](#custom-dev-server-port "Direct link to Custom Dev Server Port")

You have access to all of the [Vite dev server options](https://vitejs.dev/config/server-options.html) in your custom Vite config. You can change the dev server port by setting the `port` option.

-   JavaScript
-   TypeScript

vite.config.js

```
    export default {  server: {    port: 4000,  },}
```

.env.server

```
    WASP_WEB_CLIENT_URL=http://localhost:4000
```

vite.config.ts

```
    import { defineConfig } from 'vite'export default defineConfig({  server: {    port: 4000,  },})
```

.env.server

```
    WASP_WEB_CLIENT_URL=http://localhost:4000
```

Changing the dev server port

⚠️ Be careful when changing the dev server port, you'll need to update the `WASP_WEB_CLIENT_URL` env var in your `.env.server` file.

### Customising the Base Path[​](#customising-the-base-path "Direct link to Customising the Base Path")

If you, for example, want to serve the client from a different path than `/`, you can do that by customizing the `base` option.

-   JavaScript
-   TypeScript

vite.config.js

```
    export default {  base: '/my-app/',}
```

vite.config.ts

```
    import { defineConfig } from 'vite'export default defineConfig({  base: '/my-app/',})
```

---

# Creating New App with AI

Wasp comes with its own AI: Wasp AI, aka Mage (**M**agic web **A**pp **GE**nerator).

Wasp AI allows you to create a new Wasp app **from only a title and a short description** (using GPT in the background)!

There are two main ways to create a new Wasp app with Wasp AI:

1.  Free, open-source online app [usemage.ai](https://usemage.ai).
2.  Running `wasp new` on your machine and picking AI generation. For this you need to provide your own OpenAI API keys, but it allows for more flexibility (choosing GPT models).

They both use the same logic in the background, so both approaches are equally "smart", the difference is just in the UI / settings.

info

Wasp AI is an experimental feature. Apps that Wasp AI generates can have mistakes (proportional to their complexity), but even then they can often serve as a great starting point (once you fix the mistakes) or an interesting way to explore how to implement stuff in Wasp.

## usemage.ai[​](#usemageai "Direct link to usemage.ai")

![](/img/gpt-wasp/how-it-works.gif)

1\. Describe your app 2. Pick the color 3. Generate your app 🚀

[Mage](https://usemage.ai) is an open-source app with which you can create new Wasp apps from just a short title and description.

It is completely free for you - it uses our OpenAI API keys and we take on the costs.

Once you provide an app title, app description, and choose some basic settings, your new Wasp app will be created for you in a matter of minutes and you will be able to download it to your machine and keep working on it!

If you want to know more, check this [blog post](/blog/2023/07/10/gpt-web-app-generator) for more details on how Mage works, or this [blog post](/blog/2023/07/17/how-we-built-gpt-web-app-generator) for a high-level overview of how we implemented it.

## Wasp CLI[​](#wasp-cli "Direct link to Wasp CLI")

You can create a new Wasp app using Wasp AI by running `wasp new` in your terminal and picking AI generation.

If you don't have them set yet, `wasp` will ask you to provide (via ENV vars) your OpenAI API keys (which it will use to query GPT).

Then, after providing a title and description for your Wasp app, the new app will be generated on your disk!

![wasp-cli-ai-input](/assets/images/wasp-ai-1-a07955444bb0f6da3a6beb529b86d388.png) ![wasp-cli-ai-generation](/assets/images/wasp-ai-2-ee9892b880dc6e7c51fbe352748dbb90.png)

---

# Developing Existing App with AI

While Wasp AI doesn't at the moment offer any additional help for developing your Wasp app with AI beyond initial generation, this is something we are exploring actively.

In the meantime, while waiting for Wasp AI to add support for this, we suggest checking out [aider](https://github.com/paul-gauthier/aider), which is an AI pair programming tool in your terminal. This is a third-party tool, not affiliated with Wasp in any way, but we and some of Wasp users have found that it can be helpful when working on Wasp apps.

---

# Sending Emails

With Wasp's email-sending feature, you can easily integrate email functionality into your web application.

-   JavaScript
-   TypeScript

main.wasp

```
    app Example {  ...  emailSender: {    provider: <provider>,    defaultFrom: {      name: "Example",      email: "hello@itsme.com"    },  }}
```

main.wasp

```
    app Example {  ...  emailSender: {    provider: <provider>,    defaultFrom: {      name: "Example",      email: "hello@itsme.com"    },  }}
```

Choose from one of the providers:

-   `Dummy` (development only),
-   `Mailgun`,
-   `SendGrid`
-   or the good old `SMTP`.

Optionally, define the `defaultFrom` field, so you don't need to provide it whenever sending an email.

## Sending Emails[​](#sending-emails-1 "Direct link to Sending Emails")

Before jumping into details about setting up various providers, let's see how easy it is to send emails.

You import the `emailSender` that is provided by the `wasp/server/email` module and call the `send` method on it.

-   JavaScript
-   TypeScript

src/actions/sendEmail.js

```
    import { emailSender } from "wasp/server/email";// In some action handler...const info = await emailSender.send({  from: {    name: "John Doe",    email: "john@doe.com",  },  to: "user@domain.com",  subject: "Saying hello",  text: "Hello world",  html: "Hello <strong>world</strong>",});
```

src/actions/sendEmail.ts

```
    import { emailSender } from "wasp/server/email";// In some action handler...const info = await emailSender.send({  from: {    name: "John Doe",    email: "john@doe.com",  },  to: "user@domain.com",  subject: "Saying hello",  text: "Hello world",  html: "Hello <strong>world</strong>",});
```

Read more about the `send` method in the [API Reference](#javascript-api).

The `send` method returns an object with the status of the sent email. It varies depending on the provider you use.

## Providers[​](#providers "Direct link to Providers")

We'll go over all of the available providers in the next section. For some of them, you'll need to set up some env variables. You can do that in the `.env.server` file.

### Using the Dummy Provider[​](#using-the-dummy-provider "Direct link to Using the Dummy Provider")

Dummy Provider is not for production use

The `Dummy` provider is not for production use. It is only meant to be used during development. If you try building your app with the `Dummy` provider, the build will fail.

To speed up development, Wasp offers a `Dummy` email sender that `console.log`s the emails in the console. Since it doesn't send emails for real, it doesn't require any setup.

Set the provider to `Dummy` in your `main.wasp` file.

-   JavaScript
-   TypeScript

main.wasp

```
    app Example {  ...  emailSender: {    provider: Dummy,  }}
```

main.wasp

```
    app Example {  ...  emailSender: {    provider: Dummy,  }}
```

### Using the SMTP Provider[​](#using-the-smtp-provider "Direct link to Using the SMTP Provider")

First, set the provider to `SMTP` in your `main.wasp` file.

-   JavaScript
-   TypeScript

main.wasp

```
    app Example {  ...  emailSender: {    provider: SMTP,  }}
```

main.wasp

```
    app Example {  ...  emailSender: {    provider: SMTP,  }}
```

Then, add the following env variables to your `.env.server` file.

.env.server

```
    SMTP_HOST=SMTP_USERNAME=SMTP_PASSWORD=SMTP_PORT=
```

Many transactional email providers (e.g. Mailgun, SendGrid but also others) can also use SMTP, so you can use them as well.

### Using the Mailgun Provider[​](#using-the-mailgun-provider "Direct link to Using the Mailgun Provider")

Set the provider to `Mailgun` in the `main.wasp` file.

-   JavaScript
-   TypeScript

main.wasp

```
    app Example {  ...  emailSender: {    provider: Mailgun,  }}
```

main.wasp

```
    app Example {  ...  emailSender: {    provider: Mailgun,  }}
```

Then, get the Mailgun API key and domain and add them to your `.env.server` file.

#### Getting the API Key and Domain[​](#getting-the-api-key-and-domain "Direct link to Getting the API Key and Domain")

1.  Go to [Mailgun](https://www.mailgun.com/) and create an account.
2.  Go to [API Keys](https://app.mailgun.com/app/account/security/api_keys) and create a new API key.
3.  Copy the API key and add it to your `.env.server` file.
4.  Go to [Domains](https://app.mailgun.com/mg/sending/domains) and create a new domain.
5.  Copy the domain and add it to your `.env.server` file.

.env.server

```
    MAILGUN_API_KEY=MAILGUN_DOMAIN=
```

### Using the SendGrid Provider[​](#using-the-sendgrid-provider "Direct link to Using the SendGrid Provider")

Set the provider field to `SendGrid` in your `main.wasp` file.

-   JavaScript
-   TypeScript

main.wasp

```
    app Example {  ...  emailSender: {    provider: SendGrid,  }}
```

main.wasp

```
    app Example {  ...  emailSender: {    provider: SendGrid,  }}
```

Then, get the SendGrid API key and add it to your `.env.server` file.

#### Getting the API Key[​](#getting-the-api-key "Direct link to Getting the API Key")

1.  Go to [SendGrid](https://sendgrid.com/) and create an account.
2.  Go to [API Keys](https://app.sendgrid.com/settings/api_keys) and create a new API key.
3.  Copy the API key and add it to your `.env.server` file.

.env.server

```
    SENDGRID_API_KEY=
```

## API Reference[​](#api-reference "Direct link to API Reference")

### `emailSender` dict[​](#emailsender-dict "Direct link to emailsender-dict")

-   JavaScript
-   TypeScript

main.wasp

```
    app Example {  ...  emailSender: {    provider: <provider>,    defaultFrom: {      name: "Example",      email: "hello@itsme.com"    },  }}
```

main.wasp

```
    app Example {  ...  emailSender: {    provider: <provider>,    defaultFrom: {      name: "Example",      email: "hello@itsme.com"    },  }}
```

The `emailSender` dict has the following fields:

-   `provider: Provider` required
    
    The provider you want to use. Choose from `Dummy`, `SMTP`, `Mailgun` or `SendGrid`.
    
    Dummy Provider is not for production use
    
    The `Dummy` provider is not for production use. It is only meant to be used during development. If you try building your app with the `Dummy` provider, the build will fail.
    
-   `defaultFrom: dict`
    
    The default sender's details. If you set this field, you don't need to provide the `from` field when sending an email.
    

### JavaScript API[​](#javascript-api "Direct link to JavaScript API")

Using the `emailSender` in :

-   JavaScript
-   TypeScript

src/actions/sendEmail.js

```
    import { emailSender } from "wasp/server/email";// In some action handler...const info = await emailSender.send({  from: {    name: "John Doe",    email: "john@doe.com",  },  to: "user@domain.com",  subject: "Saying hello",  text: "Hello world",  html: "Hello <strong>world</strong>",});
```

src/actions/sendEmail.ts

```
    import { emailSender } from "wasp/server/email";// In some action handler...const info = await emailSender.send({  from: {    name: "John Doe",    email: "john@doe.com",  },  to: "user@domain.com",  subject: "Saying hello",  text: "Hello world",  html: "Hello <strong>world</strong>",});
```

The `send` method accepts an object with the following fields:

-   `from: object`
    
    The sender's details. If you set up `defaultFrom` field in the `emailSender` dict in Wasp file, this field is optional.
    
    -   `name: string`
        
        The name of the sender.
        
    -   `email: string`
        
        The email address of the sender.
        
-   `to: string` required
    
    The recipient's email address.
    
-   `subject: string` required
    
    The subject of the email.
    
-   `text: string` required
    
    The text version of the email.
    
-   `html: string` required
    
    The HTML version of the email

---

# Recurring Jobs

In most web apps, users send requests to the server and receive responses with some data. When the server responds quickly, the app feels responsive and smooth.

What if the server needs extra time to fully process the request? This might mean sending an email or making a slow HTTP request to an external API. In that case, it's a good idea to respond to the user as soon as possible and do the remaining work in the background.

Wasp supports background jobs that can help you with this:

-   Jobs persist between server restarts,
-   Jobs can be retried if they fail,
-   Jobs can be delayed until a future time,
-   Jobs can have a recurring schedule.

## Using Jobs[​](#using-jobs "Direct link to Using Jobs")

### Job Definition and Usage[​](#job-definition-and-usage "Direct link to Job Definition and Usage")

Let's write an example Job that will print a message to the console and return a list of tasks from the database.

1.  Start by creating a Job declaration in your `.wasp` file:
    
    -   JavaScript
    -   TypeScript
    
    main.wasp
    
    ```
        job mySpecialJob {  executor: PgBoss,  perform: {    fn: import { foo } from "@src/workers/bar"  },  entities: [Task],}
    ```
    
    main.wasp
    
    ```
        job mySpecialJob {  executor: PgBoss,  perform: {    fn: import { foo } from "@src/workers/bar"  },  entities: [Task],}
    ```
    
2.  After declaring the Job, implement its worker function:
    
    -   JavaScript
    -   TypeScript
    
    src/workers/bar.js
    
    ```
        export const foo = async ({ name }, context) => {  console.log(`Hello ${name}!`)  const tasks = await context.entities.Task.findMany({})  return { tasks }}
    ```
    
    src/workers/bar.ts
    
    ```
        import { type MySpecialJob } from 'wasp/server/jobs'import { type Task } from 'wasp/entities'type Input = { name: string; }type Output = { tasks: Task[]; }export const foo: MySpecialJob<Input, Output> = async ({ name }, context) => {  console.log(`Hello ${name}!`)  const tasks = await context.entities.Task.findMany({})  return { tasks }}
    ```
    
    The worker function
    
    The worker function must be an `async` function. The function's return value represents the Job's result.
    
    The worker function accepts two arguments:
    
    -   `args`: The data passed into the job when it's submitted.
    -   `context: { entities }`: The context object containing entities you put in the Job declaration.
    
3.  After successfully defining the job, you can submit work to be done in your [Operations](/docs/data-model/operations/overview) or [setupFn](/docs/project/server-config#setup-function) (or any other NodeJS code):
    
    -   JavaScript
    -   TypeScript
    
    someAction.js
    
    ```
        import { mySpecialJob } from 'wasp/server/jobs'const submittedJob = await mySpecialJob.submit({ job: "Johnny" })// Or, if you'd prefer it to execute in the future, just add a .delay().// It takes a number of seconds, Date, or ISO date string.await mySpecialJob  .delay(10)  .submit({ name: "Johnny" })
    ```
    
    someAction.ts
    
    ```
        import { mySpecialJob } from 'wasp/server/jobs'const submittedJob = await mySpecialJob.submit({ job: "Johnny" })// Or, if you'd prefer it to execute in the future, just add a .delay().// It takes a number of seconds, Date, or ISO date string.await mySpecialJob  .delay(10)  .submit({ name: "Johnny" })
    ```
    

And that's it. Your job will be executed by `PgBoss` as if you called `foo({ name: "Johnny" })`.

In our example, `foo` takes an argument, but passing arguments to jobs is not a requirement. It depends on how you've implemented your worker function.

### Recurring Jobs[​](#recurring-jobs "Direct link to Recurring Jobs")

If you have work that needs to be done on some recurring basis, you can add a `schedule` to your job declaration:

-   JavaScript
-   TypeScript

main.wasp

```
    job mySpecialJob {  executor: PgBoss,  perform: {    fn: import { foo } from "@src/workers/bar"  },  schedule: {    cron: "0 * * * *",    args: {=json { "job": "args" } json=} // optional  }}
```

main.wasp

```
    job mySpecialJob {  executor: PgBoss,  perform: {    fn: import { foo } from "@src/workers/bar"  },  schedule: {    cron: "0 * * * *",    args: {=json { "job": "args" } json=} // optional  }}
```

In this example, you *don't* need to invoke anything in . You can imagine `foo({ job: "args" })` getting automatically scheduled and invoked for you every hour.

## API Reference[​](#api-reference "Direct link to API Reference")

### Declaring Jobs[​](#declaring-jobs "Direct link to Declaring Jobs")

-   JavaScript
-   TypeScript

main.wasp

```
    job mySpecialJob {  executor: PgBoss,  perform: {    fn: import { foo } from "@src/workers/bar",    executorOptions: {      pgBoss: {=json { "retryLimit": 1 } json=}    }  },  schedule: {    cron: "*/5 * * * *",    args: {=json { "foo": "bar" } json=},    executorOptions: {      pgBoss: {=json { "retryLimit": 0 } json=}    }  },  entities: [Task],}
```

main.wasp

```
    job mySpecialJob {  executor: PgBoss,  perform: {    fn: import { foo } from "@src/workers/bar",    executorOptions: {      pgBoss: {=json { "retryLimit": 1 } json=}    }  },  schedule: {    cron: "*/5 * * * *",    args: {=json { "foo": "bar" } json=},    executorOptions: {      pgBoss: {=json { "retryLimit": 0 } json=}    }  },  entities: [Task],}
```

The Job declaration has the following fields:

-   `executor: JobExecutor` required
    
    Job executors
    
    Our jobs need job executors to handle the *scheduling, monitoring, and execution*.
    
    `PgBoss` is currently our only job executor, and is recommended for low-volume production use cases. It requires that your database provider is set to `"postgresql"` in your `schema.prisma` file. Read more about setting the provider [here](/docs/data-model/backends#postgresql).
    
    We have selected [pg-boss](https://github.com/timgit/pg-boss/) as our first job executor to handle the low-volume, basic job queue workloads many web applications have. By using PostgreSQL (and [SKIP LOCKED](https://www.2ndquadrant.com/en/blog/what-is-select-skip-locked-for-in-postgresql-9-5/)) as its storage and synchronization mechanism, it allows us to provide many job queue pros without any additional infrastructure or complex management.
    
    info
    
    Keep in mind that pg-boss jobs run alongside your other server-side code, so they are not appropriate for CPU-heavy workloads. Additionally, some care is required if you modify scheduled jobs. Please see pg-boss details below for more information.
    
    pg-boss details
    
    pg-boss provides many useful features, which can be found [here](https://github.com/timgit/pg-boss/blob/8.4.2/README.md).
    
    When you add pg-boss to a Wasp project, it will automatically add a new schema to your database called `pgboss` with some internal tracking tables, including `job` and `schedule`. pg-boss tables have a `name` column in most tables that will correspond to your Job identifier. Additionally, these tables maintain arguments, states, return values, retry information, start and expiration times, and other metadata required by pg-boss.
    
    If you need to customize the creation of the pg-boss instance, you can set an environment variable called `PG_BOSS_NEW_OPTIONS` to a stringified JSON object containing [these initialization parameters](https://github.com/timgit/pg-boss/blob/8.4.2/docs/readme.md#newoptions). **NOTE**: Setting this overwrites all Wasp defaults, so you must include database connection information as well.
    
    ### pg-boss considerations[​](#pg-boss-considerations "Direct link to pg-boss considerations")
    
    -   Wasp starts pg-boss alongside your web server's application, where both are simultaneously operational. This means that jobs running via pg-boss and the rest of the server logic (like Operations) share the CPU, therefore you should avoid running CPU-intensive tasks via jobs.
        -   Wasp does not (yet) support independent, horizontal scaling of pg-boss-only applications, nor starting them as separate workers/processes/threads.
    -   The job name/identifier in your `.wasp` file is the same name that will be used in the `name` column of pg-boss tables. If you change a name that had a `schedule` associated with it, pg-boss will continue scheduling those jobs but they will have no handlers associated, and will thus become stale and expire. To resolve this, you can remove the applicable row from the `schedule` table in the `pgboss` schema of your database.
        -   If you remove a `schedule` from a job, you will need to do the above as well.
    -   If you wish to deploy to Heroku, you need to set an additional environment variable called `PG_BOSS_NEW_OPTIONS` to `{"connectionString":"<REGULAR_HEROKU_DATABASE_URL>","ssl":{"rejectUnauthorized":false}}`. This is because pg-boss uses the `pg` extension, which does not seem to connect to Heroku over SSL by default, which Heroku requires. Additionally, Heroku uses a self-signed cert, so we must handle that as well.
    -   [https://devcenter.heroku.com/articles/connecting-heroku-postgres#connecting-in-node-js](https://devcenter.heroku.com/articles/connecting-heroku-postgres#connecting-in-node-js)
    
-   `perform: dict` required
    
    -   `fn: ExtImport` required
        
        -   An `async` function that performs the work. Since Wasp executes Jobs on the server, the import path must lead to a NodeJS file.
        -   It receives the following arguments:
            -   `args: Input`: The data passed to the job when it's submitted.
            -   `context: { entities: Entities }`: The context object containing any declared entities.
        
        Here's an example of a `perform.fn` function:
        
        -   JavaScript
        -   TypeScript
        
        src/workers/bar.js
        
        ```
            export const foo = async ({ name }, context) => {  console.log(`Hello ${name}!`)  const tasks = await context.entities.Task.findMany({})  return { tasks }}
        ```
        
        src/workers/bar.ts
        
        ```
            import { type MySpecialJob } from 'wasp/server/jobs'type Input = { name: string; }type Output = { tasks: Task[]; }export const foo: MySpecialJob<Input, Output> = async (args, context) => {  console.log(`Hello ${name}!`)  const tasks = await context.entities.Task.findMany({})  return { tasks }}
        ```
        
        Read more about type-safe jobs in the [Javascript API section](#javascript-api).
        
    -   `executorOptions: dict`
        
        Executor-specific default options to use when submitting jobs. These are passed directly through and you should consult the documentation for the job executor. These can be overridden during invocation with `submit()` or in a `schedule`.
        
        -   `pgBoss: JSON`
            
            See the docs for [pg-boss](https://github.com/timgit/pg-boss/blob/8.4.2/docs/readme.md#sendname-data-options).
            
-   `schedule: dict`
    
    -   `cron: string` required
        
        A 5-placeholder format cron expression string. See rationale for minute-level precision [here](https://github.com/timgit/pg-boss/blob/8.4.2/docs/readme.md#scheduling).
        
        *If you need help building cron expressions, Check out* *[Crontab guru](https://crontab.guru/#0_*_*_*_*).*
        
    -   `args: JSON`
        
        The arguments to pass to the `perform.fn` function when invoked.
        
    -   `executorOptions: dict`
        
        Executor-specific options to use when submitting jobs. These are passed directly through and you should consult the documentation for the job executor. The `perform.executorOptions` are the default options, and `schedule.executorOptions` can override/extend those.
        
        -   `pgBoss: JSON`
            
            See the docs for [pg-boss](https://github.com/timgit/pg-boss/blob/8.4.2/docs/readme.md#sendname-data-options).
            
-   `entities: [Entity]`
    
    A list of entities you wish to use inside your Job (similar to [Queries and Actions](/docs/data-model/operations/queries#using-entities-in-queries)).
    

### JavaScript API[​](#javascript-api "Direct link to JavaScript API")

-   Importing a Job:
    
    -   JavaScript
    -   TypeScript
    
    someAction.js
    
    ```
        import { mySpecialJob } from 'wasp/server/jobs'
    ```
    
    someAction.ts
    
    ```
        import { mySpecialJob, type MySpecialJob } from 'wasp/server/jobs'
    ```
    
    Type-safe jobs
    
    Wasp generates a generic type for each Job declaration, which you can use to type your `perform.fn` function. The type is named after the job declaration, and is available in the `wasp/server/jobs` module. In the example above, the type is `MySpecialJob`.
    
    The type takes two type arguments:
    
    -   `Input`: The type of the `args` argument of the `perform.fn` function.
    -   `Output`: The type of the return value of the `perform.fn` function.
    
-   `submit(jobArgs, executorOptions)`
    
    -   `jobArgs: Input`
        
    -   `executorOptions: object`
        
        Submits a Job to be executed by an executor, optionally passing in a JSON job argument your job handler function receives, and executor-specific submit options.
        
    
    -   JavaScript
    -   TypeScript
    
    someAction.js
    
    ```
        const submittedJob = await mySpecialJob.submit({ job: "args" })
    ```
    
    someAction.ts
    
    ```
        const submittedJob = await mySpecialJob.submit({ job: "args" })
    ```
    
-   `delay(startAfter)`
    
    -   `startAfter: int | string | Date` required
        
        Delaying the invocation of the job handler. The delay can be one of:
        
        -   Integer: number of seconds to delay. \[Default 0\]
        -   String: ISO date string to run at.
        -   Date: Date to run at.
    
    -   JavaScript
    -   TypeScript
    
    someAction.js
    
    ```
        const submittedJob = await mySpecialJob  .delay(10)  .submit({ job: "args" }, { "retryLimit": 2 })
    ```
    
    someAction.ts
    
    ```
        const submittedJob = await mySpecialJob  .delay(10)  .submit({ job: "args" }, { "retryLimit": 2 })
    ```
    

#### Tracking[​](#tracking "Direct link to Tracking")

The return value of `submit()` is an instance of `SubmittedJob`, which has the following fields:

-   `jobId`: The ID for the job in that executor.
-   `jobName`: The name of the job you used in your `.wasp` file.
-   `executorName`: The Symbol of the name of the job executor.

There are also some namespaced, job executor-specific objects.

-   For pg-boss, you may access: `pgBoss`
    -   `details()`: pg-boss specific job detail information. [Reference](https://github.com/timgit/pg-boss/blob/8.4.2/docs/readme.md#getjobbyidid)
    -   `cancel()`: attempts to cancel a job. [Reference](https://github.com/timgit/pg-boss/blob/8.4.2/docs/readme.md#cancelid)
    -   `resume()`: attempts to resume a canceled job. [Reference](https://github.com/timgit/pg-boss/blob/8.4.2/docs/readme.md#resumeid)

---

# Web Sockets

Wasp provides a fully integrated WebSocket experience by utilizing [Socket.IO](https://socket.io/) on the client and server.

We handle making sure your URLs are correctly setup, CORS is enabled, and provide a useful `useSocket` and `useSocketListener` abstractions for use in React components.

To get started, you need to:

1.  Define your WebSocket logic on the server.
2.  Enable WebSockets in your Wasp file, and connect it with your server logic.
3.  Use WebSockets on the client, in React, via `useSocket` and `useSocketListener`.
4.  Optionally, type the WebSocket events and payloads for full-stack type safety.

Let's go through setting up WebSockets step by step, starting with enabling WebSockets in your Wasp file.

## Turn On WebSockets in Your Wasp File[​](#turn-on-websockets-in-your-wasp-file "Direct link to Turn On WebSockets in Your Wasp File")

We specify that we are using WebSockets by adding `webSocket` to our `app` and providing the required `fn`. You can optionally change the auto-connect behavior.

-   JavaScript
-   TypeScript

todoApp.wasp

```
    app todoApp {  // ...  webSocket: {    fn: import { webSocketFn } from "@src/webSocket",    autoConnect: true, // optional, default: true  },}
```

todoApp.wasp

```
    app todoApp {  // ...  webSocket: {    fn: import { webSocketFn } from "@src/webSocket",    autoConnect: true, // optional, default: true  },}
```

## Defining the Events Handler[​](#defining-the-events-handler "Direct link to Defining the Events Handler")

Let's define the WebSockets server with all of the events and handler functions.

### `webSocketFn` Function[​](#websocketfn-function "Direct link to websocketfn-function")

On the server, you will get Socket.IO `io: Server` argument and `context` for your WebSocket function. The `context` object give you access to all of the entities from your Wasp app.

You can use this `io` object to register callbacks for all the regular [Socket.IO events](https://socket.io/docs/v4/server-api/). Also, if a user is logged in, you will have a `socket.data.user` on the server.

This is how we can define our `webSocketFn` function:

-   JavaScript
-   TypeScript

src/webSocket.js

```
    import { v4 as uuidv4 } from 'uuid'import { getFirstProviderUserId } from 'wasp/auth'export const webSocketFn = (io, context) => {  io.on('connection', (socket) => {    const username = getFirstProviderUserId(socket.data.user) ?? 'Unknown'    console.log('a user connected: ', username)    socket.on('chatMessage', async (msg) => {      console.log('message: ', msg)      io.emit('chatMessage', { id: uuidv4(), username, text: msg })      // You can also use your entities here:      // await context.entities.SomeEntity.create({ someField: msg })    })  })}
```

src/webSocket.ts

```
    import { v4 as uuidv4 } from 'uuid'import { getFirstProviderUserId } from 'wasp/auth'import { type WebSocketDefinition, type WaspSocketData } from 'wasp/server/webSocket'export const webSocketFn: WebSocketFn = (io, context) => {  io.on('connection', (socket) => {    const username = getFirstProviderUserId(socket.data.user) ?? 'Unknown'    console.log('a user connected: ', username)    socket.on('chatMessage', async (msg) => {      console.log('message: ', msg)      io.emit('chatMessage', { id: uuidv4(), username, text: msg })      // You can also use your entities here:      // await context.entities.SomeEntity.create({ someField: msg })    })  })}// Typing our WebSocket function with the events and payloads// allows us to get type safety on the client as welltype WebSocketFn = WebSocketDefinition<  ClientToServerEvents,  ServerToClientEvents,  InterServerEvents,  SocketData>interface ServerToClientEvents {  chatMessage: (msg: { id: string, username: string, text: string }) => void;}interface ClientToServerEvents {  chatMessage: (msg: string) => void;}interface InterServerEvents {}// Data that is attached to the socket.// NOTE: Wasp automatically injects the JWT into the connection,// and if present/valid, the server adds a user to the socket.interface SocketData extends WaspSocketData {}
```

## Using the WebSocket On The Client[​](#using-the-websocket-on-the-client "Direct link to Using the WebSocket On The Client")

### The `useSocket` Hook[​](#the-usesocket-hook "Direct link to the-usesocket-hook")

Client access to WebSockets is provided by the `useSocket` hook. It returns:

-   `socket: Socket` for sending and receiving events.
-   `isConnected: boolean` for showing a display of the Socket.IO connection status.
    -   Note: Wasp automatically connects and establishes a WebSocket connection from the client to the server by default, so you do not need to explicitly `socket.connect()` or `socket.disconnect()`.
    -   If you set `autoConnect: false` in your Wasp file, then you should call these as needed.

All components using `useSocket` share the same underlying `socket`.

### The `useSocketListener` Hook[​](#the-usesocketlistener-hook "Direct link to the-usesocketlistener-hook")

Additionally, there is a `useSocketListener: (event, callback) => void` hook which is used for registering event handlers. It takes care of unregistering the handler on unmount.

-   JavaScript
-   TypeScript

src/ChatPage.jsx

```
    import React, { useState } from 'react'import {  useSocket,  useSocketListener,} from 'wasp/client/webSocket'export const ChatPage = () => {  const [messageText, setMessageText] = useState('')  const [messages, setMessages] = useState([])  const { socket, isConnected } = useSocket()  useSocketListener('chatMessage', logMessage)  function logMessage(msg) {    setMessages((priorMessages) => [msg, ...priorMessages])  }  function handleSubmit(e) {    e.preventDefault()    socket.emit('chatMessage', messageText)    setMessageText('')  }  const messageList = messages.map((msg) => (    <li key={msg.id}>      <em>{msg.username}</em>: {msg.text}    </li>  ))  const connectionIcon = isConnected ? '🟢' : '🔴'  return (    <>      <h2>Chat {connectionIcon}</h2>      <div>        <form onSubmit={handleSubmit}>          <div>            <div>              <input                type="text"                value={messageText}                onChange={(e) => setMessageText(e.target.value)}              />            </div>            <div>              <button type="submit">Submit</button>            </div>          </div>        </form>        <ul>{messageList}</ul>      </div>    </>  )}
```

Wasp's **full-stack type safety** kicks in here: all the event types and payloads are automatically inferred from the server and are available on the client.

You can additionally use the `ClientToServerPayload` and `ServerToClientPayload` helper types to get the payload type for a specific event.

src/ChatPage.tsx

```
    import React, { useState } from 'react'import {  useSocket,  useSocketListener,  ServerToClientPayload,} from 'wasp/client/webSocket'export const ChatPage = () => {  const [messageText, setMessageText] = useState<    // We are using a helper type to get the payload type for the "chatMessage" event.    ClientToServerPayload<'chatMessage'>  >('')  const [messages, setMessages] = useState<    ServerToClientPayload<'chatMessage'>[]  >([])  // The "socket" instance is typed with the types you defined on the server.  const { socket, isConnected } = useSocket()  // This is a type-safe event handler: "chatMessage" event and its payload type  // are defined on the server.  useSocketListener('chatMessage', logMessage)  function logMessage(msg: ServerToClientPayload<'chatMessage'>) {    setMessages((priorMessages) => [msg, ...priorMessages])  }  function handleSubmit(e: React.FormEvent<HTMLFormElement>) {    e.preventDefault()    // This is a type-safe event emitter: "chatMessage" event and its payload type    // are defined on the server.    socket.emit('chatMessage', messageText)    setMessageText('')  }  const messageList = messages.map((msg) => (    <li key={msg.id}>      <em>{msg.username}</em>: {msg.text}    </li>  ))  const connectionIcon = isConnected ? '🟢' : '🔴'  return (    <>      <h2>Chat {connectionIcon}</h2>      <div>        <form onSubmit={handleSubmit}>          <div>            <div>              <input                type="text"                value={messageText}                onChange={(e) => setMessageText(e.target.value)}              />            </div>            <div>              <button type="submit">Submit</button>            </div>          </div>        </form>        <ul>{messageList}</ul>      </div>    </>  )}
```

## API Reference[​](#api-reference "Direct link to API Reference")

-   JavaScript
-   TypeScript

todoApp.wasp

```
    app todoApp {  // ...  webSocket: {    fn: import { webSocketFn } from "@src/webSocket",    autoConnect: true, // optional, default: true  },}
```

todoApp.wasp

```
    app todoApp {  // ...  webSocket: {    fn: import { webSocketFn } from "@src/webSocket",    autoConnect: true, // optional, default: true  },}
```

The `webSocket` dict has the following fields:

-   `fn: WebSocketFn` required
    
    The function that defines the WebSocket events and handlers.
    
-   `autoConnect: bool`
    
    Whether to automatically connect to the WebSocket server. Default: `true`.

---

# Accessing the configuration

Whenever you start a Wasp app, you are starting two processes.

-   **The client process** - A React app that implements your app's frontend.
    
    During development, this is a dev server with hot reloading. In production, it's a simple process that serves pre-built static files with environment variables embedded during the build (details depend on [how you deploy it](/docs/advanced/deployment/overview)).
    
-   **The server process** - An Express server that implements your app's backend.
    
    During development, this is an Express server controlled by a [`nodemon`](https://www.npmjs.com/package/nodemon) process that takes care of hot reloading and restarts. In production, it's a regular Express server run using Node.
    

Check [the introduction](/docs) for a more in-depth explanation of Wasp's runtime architecture.

You can configure both processes through environment variables. See [the deployment instructions](/docs/advanced/deployment/manually#environment-variables) for a full list of supported variables.

Wasp gives you runtime access to the processes' configurations through **configuration objects**.

## Server configuration object[​](#server-configuration-object "Direct link to Server configuration object")

The server configuration object contains these fields:

-   `frontendUrl: String` - Set it with env var `WASP_WEB_CLIENT_URL`.
    
    The URL of your client (the app's frontend).  
    Wasp automatically sets it during development when you run `wasp start`.  
    In production, you should set it to your client's URL as the server sees it (i.e., with the DNS and proxies considered).
    

You can access it like this:

```
    import { config } from 'wasp/server'console.log(config.frontendUrl)
```

## Client configuration object[​](#client-configuration-object "Direct link to Client configuration object")

The client configuration object contains these fields:

-   `apiUrl: String` - Set it with env var `REACT_APP_API_URL`
    
    The URL of your server (the app's backend).  
    Wasp automatically sets it during development when you run `wasp start`.  
    In production, it should contain the value of your server's URL as the user's browser sees it (i.e., with the DNS and proxies considered).
    

You can access it like this:

```
    import { config } from 'wasp/client'console.log(config.apiUrl)
```

---

# Custom HTTP API Endpoints

In Wasp, the default client-server interaction mechanism is through [Operations](/docs/data-model/operations/overview). However, if you need a specific URL method/path, or a specific response, Operations may not be suitable for you. For these cases, you can use an `api`. Best of all, they should look and feel very familiar.

## How to Create an API[​](#how-to-create-an-api "Direct link to How to Create an API")

APIs are used to tie a JS function to a certain endpoint e.g. `POST /something/special`. They are distinct from Operations and have no client-side helpers (like `useQuery`).

To create a Wasp API, you must:

1.  Declare the API in Wasp using the `api` declaration
2.  Define the API's NodeJS implementation

After completing these two steps, you'll be able to call the API from the client code (via our `Axios` wrapper), or from the outside world.

### Declaring the API in Wasp[​](#declaring-the-api-in-wasp "Direct link to Declaring the API in Wasp")

First, we need to declare the API in the Wasp file and you can easily do this with the `api` declaration:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...api fooBar { // APIs and their implementations don't need to (but can) have the same name.  fn: import { fooBar } from "@src/apis",  httpRoute: (GET, "/foo/bar")}
```

main.wasp

```
    // ...api fooBar { // APIs and their implementations don't need to (but can) have the same name.  fn: import { fooBar } from "@src/apis",  httpRoute: (GET, "/foo/bar")}
```

Read more about the supported fields in the [API Reference](#api-reference).

### Defining the API's NodeJS Implementation[​](#defining-the-apis-nodejs-implementation "Direct link to Defining the API's NodeJS Implementation")

After you defined the API, it should be implemented as a NodeJS function that takes three arguments:

1.  `req`: Express Request object
2.  `res`: Express Response object
3.  `context`: An additional context object **injected into the API by Wasp**. This object contains user session information, as well as information about entities. The examples here won't use the context for simplicity purposes. You can read more about it in the [section about using entities in APIs](#using-entities-in-apis).

-   JavaScript
-   TypeScript

src/apis.js

```
    export const fooBar = (req, res, context) => {  res.set("Access-Control-Allow-Origin", "*"); // Example of modifying headers to override Wasp default CORS middleware.  res.json({ msg: `Hello, ${context.user ? "registered user" : "stranger"}!` });};
```

src/apis.ts

```
    import { FooBar } from "wasp/server/api"; // This type is generated by Wasp based on the `api` declaration above.export const fooBar: FooBar = (req, res, context) => {  res.set("Access-Control-Allow-Origin", "*"); // Example of modifying headers to override Wasp default CORS middleware.  res.json({ msg: `Hello, ${context.user ? "registered user" : "stranger"}!` });};
```

## Using the API[​](#using-the-api "Direct link to Using the API")

### Using the API externally[​](#using-the-api-externally "Direct link to Using the API externally")

To use the API externally, you simply call the endpoint using the method and path you used.

For example, if your app is running at `https://example.com` then from the above you could issue a `GET` to `https://example/com/foo/callback` (in your browser, Postman, `curl`, another web service, etc.).

### Using the API from the Client[​](#using-the-api-from-the-client "Direct link to Using the API from the Client")

To use the API from your client, including with auth support, you can import the Axios wrapper from `wasp/client/api` and invoke a call. For example:

-   JavaScript
-   TypeScript

src/pages/SomePage.jsx

```
    import React, { useEffect } from "react";import { api } from "wasp/client/api";async function fetchCustomRoute() {  const res = await api.get("/foo/bar");  console.log(res.data);}export const Foo = () => {  useEffect(() => {    fetchCustomRoute();  }, []);  return <>// ...</>;};
```

src/pages/SomePage.tsx

```
    import React, { useEffect } from "react";import { api } from "wasp/client/api";async function fetchCustomRoute() {  const res = await api.get("/foo/bar");  console.log(res.data);}export const Foo = () => {  useEffect(() => {    fetchCustomRoute();  }, []);  return <>// ...</>;};
```

#### Making Sure CORS Works[​](#making-sure-cors-works "Direct link to Making Sure CORS Works")

APIs are designed to be as flexible as possible, hence they don't utilize the default middleware like Operations do. As a result, to use these APIs on the client side, you must ensure that CORS (Cross-Origin Resource Sharing) is enabled.

You can do this by defining custom middleware for your APIs in the Wasp file.

-   JavaScript
-   TypeScript

For example, an `apiNamespace` is a simple declaration used to apply some `middlewareConfigFn` to all APIs under some specific path:

main.wasp

```
    apiNamespace fooBar {  middlewareConfigFn: import { fooBarNamespaceMiddlewareFn } from "@src/apis",  path: "/foo"}
```

And then in the implementation file:

src/apis.js

```
    export const apiMiddleware = (config) => {  return config;};
```

For example, an `apiNamespace` is a simple declaration used to apply some `middlewareConfigFn` to all APIs under some specific path:

main.wasp

```
    apiNamespace fooBar {  middlewareConfigFn: import { fooBarNamespaceMiddlewareFn } from "@src/apis",  path: "/foo"}
```

And then in the implementation file (returning the default config):

src/apis.ts

```
    import { MiddlewareConfigFn } from "wasp/server";export const apiMiddleware: MiddlewareConfigFn = (config) => {  return config;};
```

We are returning the default middleware which enables CORS for all APIs under the `/foo` path.

For more information about middleware configuration, please see: [Middleware Configuration](/docs/advanced/middleware-config)

## Using Entities in APIs[​](#using-entities-in-apis "Direct link to Using Entities in APIs")

In many cases, resources used in APIs will be [Entities](/docs/data-model/entities). To use an Entity in your API, add it to the `api` declaration in Wasp:

-   JavaScript
-   TypeScript

main.wasp

```
    api fooBar {  fn: import { fooBar } from "@src/apis",  entities: [Task],  httpRoute: (GET, "/foo/bar")}
```

main.wasp

```
    api fooBar {  fn: import { fooBar } from "@src/apis",  entities: [Task],  httpRoute: (GET, "/foo/bar")}
```

Wasp will inject the specified Entity into the APIs `context` argument, giving you access to the Entity's Prisma API:

-   JavaScript
-   TypeScript

src/apis.js

```
    export const fooBar = (req, res, context) => {  res.json({ count: await context.entities.Task.count() });};
```

src/apis.ts

```
    import { FooBar } from "wasp/server/api";export const fooBar: FooBar = (req, res, context) => {  res.json({ count: await context.entities.Task.count() });};
```

The object `context.entities.Task` exposes `prisma.task` from [Prisma's CRUD API](https://www.prisma.io/docs/reference/tools-and-interfaces/prisma-client/crud).

## API Reference[​](#api-reference "Direct link to API Reference")

-   JavaScript
-   TypeScript

main.wasp

```
    api fooBar {  fn: import { fooBar } from "@src/apis",  httpRoute: (GET, "/foo/bar"),  entities: [Task],  auth: true,  middlewareConfigFn: import { apiMiddleware } from "@src/apis"}
```

main.wasp

```
    api fooBar {  fn: import { fooBar } from "@src/apis",  httpRoute: (GET, "/foo/bar"),  entities: [Task],  auth: true,  middlewareConfigFn: import { apiMiddleware } from "@src/apis"}
```

The `api` declaration has the following fields:

-   `fn: ExtImport` required
    
    The import statement of the APIs NodeJs implementation.
    
-   `httpRoute: (HttpMethod, string)` required
    
    The HTTP (method, path) pair, where the method can be one of:
    
    -   `ALL`, `GET`, `POST`, `PUT` or `DELETE`
    -   and path is an Express path `string`.
-   `entities: [Entity]`
    
    A list of entities you wish to use inside your API. You can read more about it [here](#using-entities-in-apis).
    
-   `auth: bool`
    
    If auth is enabled, this will default to `true` and provide a `context.user` object. If you do not wish to attempt to parse the JWT in the Authorization Header, you should set this to `false`.
    
-   `middlewareConfigFn: ExtImport`
    
    The import statement to an Express middleware config function for this API. See more in [middleware section](/docs/advanced/middleware-config) of the docs.

---

# Configuring Middleware

Wasp comes with a minimal set of useful Express middleware in every application. While this is good for most users, we realize some may wish to add, modify, or remove some of these choices both globally, or on a per-`api`/path basis.

## Default Global Middleware 🌍[​](#default-global-middleware- "Direct link to Default Global Middleware 🌍")

Wasp's Express server has the following middleware by default:

-   [Helmet](https://helmetjs.github.io/): Helmet helps you secure your Express apps by setting various HTTP headers. *It's not a silver bullet, but it's a good start.*
    
-   [CORS](https://github.com/expressjs/cors#readme): CORS is a package for providing a middleware that can be used to enable [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) with various options.
    
    note
    
    CORS middleware is required for the frontend to communicate with the backend.
    
-   [Morgan](https://github.com/expressjs/morgan#readme): HTTP request logger middleware.
    
-   [express.json](https://expressjs.com/en/api.html#express.json) (which uses [body-parser](https://github.com/expressjs/body-parser#bodyparserjsonoptions)): parses incoming request bodies in a middleware before your handlers, making the result available under the `req.body` property.
    
    note
    
    JSON middleware is required for [Operations](/docs/data-model/operations/overview) to function properly.
    
-   [express.urlencoded](https://expressjs.com/en/api.html#express.urlencoded) (which uses [body-parser](https://expressjs.com/en/resources/middleware/body-parser.html#bodyparserurlencodedoptions)): returns middleware that only parses urlencoded bodies and only looks at requests where the `Content-Type` header matches the type option.
    
-   [cookieParser](https://github.com/expressjs/cookie-parser#readme): parses Cookie header and populates `req.cookies` with an object keyed by the cookie names.
    

## Customization[​](#customization "Direct link to Customization")

You have three places where you can customize middleware:

1.  [global](#1-customize-global-middleware): here, any changes will apply by default *to all operations (`query` and `action`) and `api`.* This is helpful if you wanted to add support for multiple domains to CORS, for example.
    
    Modifying global middleware
    
    Please treat modifications to global middleware with extreme care as they will affect all operations and APIs. If you are unsure, use one of the other two options.
    
2.  [per-api](#2-customize-api-specific-middleware): you can override middleware for a specific api route (e.g. `POST /webhook/callback`). This is helpful if you want to disable JSON parsing for some callback, for example.
    
3.  [per-path](#3-customize-per-path-middleware): this is helpful if you need to customize middleware for all methods under a given path.
    
    -   It's helpful for things like "complex CORS requests" which may need to apply to both `OPTIONS` and `GET`, or to apply some middleware to a *set of `api` routes*.

### Default Middleware Definitions[​](#default-middleware-definitions "Direct link to Default Middleware Definitions")

Below is the actual definitions of default middleware which you can override.

-   JavaScript
-   TypeScript

```
    const defaultGlobalMiddleware = new Map([  ['helmet', helmet()],  ['cors', cors({ origin: config.allowedCORSOrigins })],  ['logger', logger('dev')],  ['express.json', express.json()],  ['express.urlencoded', express.urlencoded({ extended: false })],  ['cookieParser', cookieParser()]])
```

```
    export type MiddlewareConfig = Map<string, express.RequestHandler>// Used in the examples below 👇export type MiddlewareConfigFn = (middlewareConfig: MiddlewareConfig) => MiddlewareConfigconst defaultGlobalMiddleware: MiddlewareConfig = new Map([  ['helmet', helmet()],  ['cors', cors({ origin: config.allowedCORSOrigins })],  ['logger', logger('dev')],  ['express.json', express.json()],  ['express.urlencoded', express.urlencoded({ extended: false })],  ['cookieParser', cookieParser()]])
```

## 1\. Customize Global Middleware[​](#1-customize-global-middleware "Direct link to 1. Customize Global Middleware")

If you would like to modify the middleware for *all* operations and APIs, you can do something like:

-   JavaScript
-   TypeScript

main.wasp

```
    app todoApp {  // ...  server: {    setupFn: import setup from "@src/serverSetup",    middlewareConfigFn: import { serverMiddlewareFn } from "@src/serverSetup"  },}
```

src/serverSetup.js

```
    import cors from 'cors'import { config } from 'wasp/server'export const serverMiddlewareFn = (middlewareConfig) => {  // Example of adding extra domains to CORS.  middlewareConfig.set('cors', cors({ origin: [config.frontendUrl, 'https://example1.com', 'https://example2.com'] }))  return middlewareConfig}
```

main.wasp

```
    app todoApp {  // ...  server: {    setupFn: import setup from "@src/serverSetup",    middlewareConfigFn: import { serverMiddlewareFn } from "@src/serverSetup"  },}
```

src/serverSetup.ts

```
    import cors from 'cors'import { config, type MiddlewareConfigFn } from 'wasp/server'export const serverMiddlewareFn: MiddlewareConfigFn = (middlewareConfig) => {  // Example of adding an extra domains to CORS.  middlewareConfig.set('cors', cors({ origin: [config.frontendUrl, 'https://example1.com', 'https://example2.com'] }))  return middlewareConfig}
```

## 2\. Customize `api`\-specific Middleware[​](#2-customize-api-specific-middleware "Direct link to 2-customize-api-specific-middleware")

If you would like to modify the middleware for a single API, you can do something like:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...api webhookCallback {  fn: import { webhookCallback } from "@src/apis",  middlewareConfigFn: import { webhookCallbackMiddlewareFn } from "@src/apis",  httpRoute: (POST, "/webhook/callback"),  auth: false}
```

src/apis.js

```
    import express from 'express'export const webhookCallback = (req, res, _context) => {  res.json({ msg: req.body.length })}export const webhookCallbackMiddlewareFn = (middlewareConfig) => {  console.log('webhookCallbackMiddlewareFn: Swap express.json for express.raw')  middlewareConfig.delete('express.json')  middlewareConfig.set('express.raw', express.raw({ type: '*/*' }))  return middlewareConfig}
```

main.wasp

```
    // ...api webhookCallback {  fn: import { webhookCallback } from "@src/apis",  middlewareConfigFn: import { webhookCallbackMiddlewareFn } from "@src/apis",  httpRoute: (POST, "/webhook/callback"),  auth: false}
```

src/apis.ts

```
    import express from 'express'import { type WebhookCallback } from 'wasp/server/api'import { type MiddlewareConfigFn } from 'wasp/server'export const webhookCallback: WebhookCallback = (req, res, _context) => {  res.json({ msg: req.body.length })}export const webhookCallbackMiddlewareFn: MiddlewareConfigFn = (middlewareConfig) => {  console.log('webhookCallbackMiddlewareFn: Swap express.json for express.raw')  middlewareConfig.delete('express.json')  middlewareConfig.set('express.raw', express.raw({ type: '*/*' }))  return middlewareConfig}
```

note

This gets installed on a per-method basis. Behind the scenes, this results in code like:

```
    router.post('/webhook/callback', webhookCallbackMiddleware, ...)
```

## 3\. Customize Per-Path Middleware[​](#3-customize-per-path-middleware "Direct link to 3. Customize Per-Path Middleware")

If you would like to modify the middleware for all API routes under some common path, you can define a `middlewareConfigFn` on an `apiNamespace`:

-   JavaScript
-   TypeScript

main.wasp

```
    // ...apiNamespace fooBar {  middlewareConfigFn: import { fooBarNamespaceMiddlewareFn } from "@src/apis",  path: "/foo/bar"}
```

src/apis.js

```
    export const fooBarNamespaceMiddlewareFn = (middlewareConfig) => {  const customMiddleware = (_req, _res, next) => {    console.log('fooBarNamespaceMiddlewareFn: custom middleware')    next()  }  middlewareConfig.set('custom.middleware', customMiddleware)  return middlewareConfig}
```

main.wasp

```
    // ...apiNamespace fooBar {  middlewareConfigFn: import { fooBarNamespaceMiddlewareFn } from "@src/apis",  path: "/foo/bar"}
```

src/apis.ts

```
    import express from 'express'import { type MiddlewareConfigFn } from 'wasp/server'export const fooBarNamespaceMiddlewareFn: MiddlewareConfigFn = (middlewareConfig) => {  const customMiddleware: express.RequestHandler = (_req, _res, next) => {    console.log('fooBarNamespaceMiddlewareFn: custom middleware')    next()  }  middlewareConfig.set('custom.middleware', customMiddleware)  return middlewareConfig}
```

note

This gets installed at the router level for the path. Behind the scenes, this results in something like:

```
    router.use('/foo/bar', fooBarNamespaceMiddleware)
```

---

# Type-Safe Links

If you are using Typescript, you can use Wasp's custom `Link` component to create type-safe links to other pages on your site.

## Using the `Link` Component[​](#using-the-link-component "Direct link to using-the-link-component")

After you defined a route:

main.wasp

```
    route TaskRoute { path: "/task/:id", to: TaskPage }page TaskPage { ... }
```

You can get the benefits of type-safe links by using the `Link` component from `wasp/client/router`:

TaskList.tsx

```
    import { Link } from 'wasp/client/router'export const TaskList = () => {  // ...  return (    <div>      {tasks.map((task) => (        <Link          key={task.id}          to="/task/:id"          {/* 👆 You must provide a valid path here */}          params={{ id: task.id }}>          {/* 👆 All the params must be correctly passed in */}          {task.description}        </Link>      ))}    </div>  )}
```

### Using Search Query & Hash[​](#using-search-query--hash "Direct link to Using Search Query & Hash")

You can also pass `search` and `hash` props to the `Link` component:

TaskList.tsx

```
    <Link  to="/task/:id"  params={{ id: task.id }}  search={{ sortBy: 'date' }}  hash="comments">  {task.description}</Link>
```

This will result in a link like this: `/task/1?sortBy=date#comments`. Check out the [API Reference](#link-component) for more details.

## The `routes` Object[​](#the-routes-object "Direct link to the-routes-object")

You can also get all the pages in your app with the `routes` object:

TaskList.tsx

```
    import { routes } from 'wasp/client/router'const linkToTask = routes.TaskRoute.build({ params: { id: 1 } })
```

This will result in a link like this: `/task/1`.

You can also pass `search` and `hash` props to the `build` function. Check out the [API Reference](#routes-object) for more details.

## API Reference[​](#api-reference "Direct link to API Reference")

### `Link` Component[​](#link-component "Direct link to link-component")

The `Link` component accepts the following props:

-   `to` required
    
    -   A valid Wasp Route path from your `main.wasp` file.
-   `params: { [name: string]: string | number }` required (if the path contains params)
    
    -   An object with keys and values for each param in the path.
    -   For example, if the path is `/task/:id`, then the `params` prop must be `{ id: 1 }`. Wasp supports required and optional params.
-   `search: string[][] | Record<string, string> | string | URLSearchParams`
    
    -   Any valid input for `URLSearchParams` constructor.
    -   For example, the object `{ sortBy: 'date' }` becomes `?sortBy=date`.
-   `hash: string`
    
-   all other props that the `react-router-dom`'s [Link](https://v5.reactrouter.com/web/api/Link) component accepts
    

### `routes` Object[​](#routes-object "Direct link to routes-object")

The `routes` object contains a function for each route in your app.

router.tsx

```
    export const routes = {  // RootRoute has a path like "/"  RootRoute: {    build: (options?: {      search?: string[][] | Record<string, string> | string | URLSearchParams      hash?: string    }) => // ...  },  // DetailRoute has a path like "/task/:id/:something?"  DetailRoute: {    build: (      options: {        params: { id: ParamValue; something?: ParamValue; },        search?: string[][] | Record<string, string> | string | URLSearchParams        hash?: string      }    ) => // ...  }}
```

The `params` object is required if the route contains params. The `search` and `hash` parameters are optional.

You can use the `routes` object like this:

```
    import { routes } from 'wasp/client/router'const linkToRoot = routes.RootRoute.build()const linkToTask = routes.DetailRoute.build({ params: { id: 1 } })
```

---

# Overview

Wasp apps are full-stack apps that consist of:

-   A Node.js server.
-   A static client.
-   A PostgreSQL database.

You can deploy each part **anywhere** where you can usually deploy Node.js apps or static apps. For example, you can deploy your client on [Netlify](https://www.netlify.com/), the server on [Fly.io](https://fly.io/), and the database on [Neon](https://neon.tech/).

To make deploying as smooth as possible, Wasp also offers a single-command deployment through the **Wasp CLI**.

[

### Using Wasp CLI »

One command deployment & redeployment

](/docs/advanced/deployment/cli)[

### Deploying Manually »

Build the app and deploy it manually

](/docs/advanced/deployment/manually)

Click on each deployment method for more details.

Regardless of how you choose to deploy your app (i.e., manually or using the Wasp CLI), you'll need to know about some common patterns covered below.

## Customizing the Dockerfile[​](#customizing-the-dockerfile "Direct link to Customizing the Dockerfile")

By default, Wasp generates a multi-stage Dockerfile. This file is used to build and run a Docker image with the Wasp-generated server code. It also runs any pending migrations.

You can **add extra steps to this multi-stage `Dockerfile`** by creating your own `Dockerfile` in the project's root directory. If Wasp finds a Dockerfile in the project's root, it appends its contents at the *bottom* of the default multi-stage Dockerfile.

Since the last definition in a Dockerfile wins, you can override or continue from any existing build stages. You can also choose not to use any of our build stages and have your own custom Dockerfile used as-is.

A few things to keep in mind:

-   If you override an intermediate build stage, no later build stages will be used unless you reproduce them below.
-   The generated Dockerfile's content is dynamic and depends on which features your app uses. The content can also change in future releases, so please verify it from time to time.
-   Make sure to supply `ENTRYPOINT` in your final build stage. Your changes won't have any effect if you don't.

Read more in the official Docker docs on [multi-stage builds](https://docs.docker.com/build/building/multi-stage/).

To see what your project's (potentially combined) Dockerfile will look like, run:

```
    wasp dockerfile
```

Join our [Discord](https://discord.gg/rzdnErX) if you have any questions, or if you need more customization than this hook provides.

---

# Deploying with the Wasp CLI

Wasp CLI can deploy your full-stack application with only a single command. The command automates the manual deployment process and is the recommended way of deploying Wasp apps.

## Supported Providers[​](#supported-providers "Direct link to Supported Providers")

Wasp supports automated deployment to the following providers:

-   [Fly.io](#flyio) - they offer 5$ free credit each month
-   Railway (coming soon, track it here [#1157](https://github.com/wasp-lang/wasp/pull/1157))

## Fly.io[​](#flyio "Direct link to Fly.io")

### Prerequisites[​](#prerequisites "Direct link to Prerequisites")

Fly provides [free allowances](https://fly.io/docs/about/pricing/#plans) for up to 3 VMs (so deploying a Wasp app to a new account is free), but all plans require you to add your credit card information before you can proceed. If you don't, the deployment will fail.

You can add the required credit card information on the [account's billing page](https://fly.io/dashboard/personal/billing).

Fly.io CLI

You will need the [`flyctl` CLI](https://fly.io/docs/hands-on/install-flyctl/) installed on your machine before you can deploy to Fly.io.

### Deploying[​](#deploying "Direct link to Deploying")

Using the Wasp CLI, you can easily deploy a new app to [Fly.io](https://fly.io) with just a single command:

```
    wasp deploy fly launch my-wasp-app mia
```

Specifying Org

If your account is a member of more than one organization on Fly.io, you will need to specify under which one you want to execute the command. To do that, provide an additional `--org <org-slug>` option. You can find out the names(slugs) of your organizations by running `fly orgs list`.

Please do not CTRL-C or exit your terminal while the commands are running.

Under the covers, this runs the equivalent of the following commands:

```
    wasp deploy fly setup my-wasp-app miawasp deploy fly create-db miawasp deploy fly deploy
```

The commands above use the app basename `my-wasp-app` and deploy it to the *Miami, Florida (US) region* (called `mia`). Read more about Fly.io regions [here](#flyio-regions).

Unique Name

Your app name must be unique across all of Fly or deployment will fail.

The basename is used to create all three app tiers, resulting in three separate apps in your Fly dashboard:

-   `my-wasp-app-client`
-   `my-wasp-app-server`
-   `my-wasp-app-db`

You'll notice that Wasp creates two new files in your project root directory:

-   `fly-server.toml`
-   `fly-client.toml`

You should include these files in your version control so that you can deploy your app with a single command in the future.

### Using a Custom Domain For Your App[​](#using-a-custom-domain-for-your-app "Direct link to Using a Custom Domain For Your App")

Setting up a custom domain is a three-step process:

1.  You need to add your domain to your Fly client app. You can do this by running:

```
    wasp deploy fly cmd --context client certs create mycoolapp.com
```

Use Your Domain

Make sure to replace `mycoolapp.com` with your domain in all of the commands mentioned in this section.

This command will output the instructions to add the DNS records to your domain. It will look something like this:

```
    You can direct traffic to mycoolapp.com by:1: Adding an A record to your DNS service which reads    A @ 66.241.1XX.154You can validate your ownership of mycoolapp.com by:2: Adding an AAAA record to your DNS service which reads:    AAAA @ 2a09:82XX:1::1:ff40
```

2.  You need to add the DNS records for your domain:
    
    *This will depend on your domain provider, but it should be a matter of adding an A record for `@` and an AAAA record for `@` with the values provided by the previous command.*
    
3.  You need to set your domain as the `WASP_WEB_CLIENT_URL` environment variable for your server app:
    

```
    wasp deploy fly cmd --context server secrets set WASP_WEB_CLIENT_URL=https://mycoolapp.com
```

We need to do this to keep our CORS configuration up to date.

That's it, your app should be available at `https://mycoolapp.com`! 🎉

#### Adding www Subdomain[​](#adding-www-subdomain "Direct link to Adding www Subdomain")

If you'd like to also access your app at `https://www.mycoolapp.com`, you can generate certs for the `www` subdomain.

```
    wasp deploy fly cmd --context client certs create www.mycoolapp.com
```

Once you do that, you will need to add another DNS record for your domain. It should be a CNAME record for `www` with the value of your root domain. Here's an example:

Type

Name

Value

TTL

CNAME

www

mycoolapp.com

3600

With the CNAME record (Canonical name), you are assigning the `www` subdomain as an alias to the root domain.

Your app should now be available both at the root domain `https://mycoolapp.com` and the `www` sub-domain `https://www.mycoolapp.com`! 🎉

## API Reference[​](#api-reference "Direct link to API Reference")

### `launch`[​](#launch "Direct link to launch")

`launch` is a convenience command that runs `setup`, `create-db`, and `deploy` in sequence.

```
    wasp deploy fly launch <app-name> <region>
```

It accepts the following arguments:

-   `<app-name>` - the name of your app required
    
-   `<region>` - the region where your app will be deployed required
    
    Read how to find the available regions [here](#flyio-regions).
    

It gives you the same result as running the following commands:

```
    wasp deploy fly setup <app-name> <region>wasp deploy fly create-db <region>wasp deploy fly deploy
```

#### Environment Variables[​](#environment-variables "Direct link to Environment Variables")

##### Server[​](#server "Direct link to Server")

If you are deploying an app that requires any other environment variables (like social auth secrets), you can set them with the `--server-secret` option:

```
    wasp deploy fly launch my-wasp-app mia --server-secret GOOGLE_CLIENT_ID=<...> --server-secret GOOGLE_CLIENT_SECRET=<...>
```

##### Client[​](#client "Direct link to Client")

If you've added any [client-side environment variables](/docs/project/env-vars#client-env-vars) to your app, make sure to pass them to the terminal session before running the `launch` command, e.g.:

```
    REACT_APP_ANOTHER_VAR=somevalue wasp deploy fly launch my-wasp-app mia
```

### `setup`[​](#setup "Direct link to setup")

`setup` will create your client and server apps on Fly, and add some secrets, but does *not* deploy them.

```
    wasp deploy fly setup <app-name> <region>
```

It accepts the following arguments:

-   `<app-name>` - the name of your app required
    
-   `<region>` - the region where your app will be deployed required
    
    Read how to find the available regions [here](#flyio-regions).
    

After running `setup`, Wasp creates two new files in your project root directory: `fly-server.toml` and `fly-client.toml`. You should include these files in your version control.

You **can edit the `fly-server.toml` and `fly-client.toml` files** to further configure your Fly deployments. Wasp will use the TOML files when you run `deploy`.

If you want to maintain multiple apps, you can add the `--fly-toml-dir <abs-path>` option to point to different directories, like "dev" or "staging".

Execute Only Once

You should only run `setup` once per app. If you run it multiple times, it will create unnecessary apps on Fly.

### `create-db`[​](#create-db "Direct link to create-db")

`create-db` will create a new database for your app.

```
    wasp deploy fly create-db <region>
```

It accepts the following arguments:

-   `<region>` - the region where your app will be deployed required
    
    Read how to find the available regions [here](#flyio-regions).
    

Execute Only Once

You should only run `create-db` once per app. If you run it multiple times, it will create multiple databases, but your app needs only one.

### `deploy`[​](#deploy "Direct link to deploy")

```
    wasp deploy fly deploy
```

`deploy` pushes your client and server live.

Run this command whenever you want to **update your deployed app** with the latest changes:

```
    wasp deploy fly deploy
```

If you've added any [client-side environment variables](/docs/project/env-vars#client-env-vars) to your app, make sure to pass them to the terminal session before running the `deploy` command, e.g.:

```
    REACT_APP_ANOTHER_VAR=somevalue wasp deploy fly deploy
```

Make sure to add your client-side environment variables every time you redeploy with the above command [to ensure they are included in the build process](/docs/project/env-vars#client-env-vars-1)!

### `cmd`[​](#cmd "Direct link to cmd")

If want to run arbitrary Fly commands (e.g. `flyctl secrets list` for your server app), here's how to do it:

```
    wasp deploy fly cmd secrets list --context server
```

### Environment Variables[​](#environment-variables-1 "Direct link to Environment Variables")

#### Server Secrets[​](#server-secrets "Direct link to Server Secrets")

If your app requires any other server-side environment variables (like social auth secrets), you can set them:

1.  initially in the `launch` command with the [`--server-secret` option](#environment-variables),  
    or
2.  after the app has already been deployed by using the `secrets set` command:

```
    wasp deploy fly cmd secrets set GOOGLE_CLIENT_ID=<...> GOOGLE_CLIENT_SECRET=<...> --context=server
```

#### Client Environment Variables[​](#client-environment-variables "Direct link to Client Environment Variables")

If you've added any [client-side environment variables](/docs/project/env-vars#client-env-vars) to your app, make sure to pass them to the terminal session before running a deployment command, e.g.:

```
    REACT_APP_ANOTHER_VAR=somevalue wasp deploy fly launch my-wasp-app mia
```

or

```
    REACT_APP_ANOTHER_VAR=somevalue wasp deploy fly deploy
```

### Fly.io Regions[​](#flyio-regions "Direct link to Fly.io Regions")

> Fly.io runs applications physically close to users: in datacenters around the world, on servers we run ourselves. You can currently deploy your apps in 34 regions, connected to a global Anycast network that makes sure your users hit our nearest server, whether they’re in Tokyo, São Paolo, or Frankfurt.

Read more on Fly regions [here](https://fly.io/docs/reference/regions/).

You can find the list of all available Fly regions by running:

```
    flyctl platform regions
```

### Multiple Fly.io Organizations[​](#multiple-flyio-organizations "Direct link to Multiple Fly.io Organizations")

If you have multiple organizations, you can specify a `--org` option. For example:

```
    wasp deploy fly launch my-wasp-app mia --org hive
```

### Building Locally[​](#building-locally "Direct link to Building Locally")

Fly.io offers support for both **locally** built Docker containers and **remotely** built ones. However, for simplicity and reproducibility, the CLI defaults to the use of a remote Fly.io builder.

If you want to build locally, supply the `--build-locally` option to `wasp deploy fly launch` or `wasp deploy fly deploy`.

---

# Deploying Manually

This document explains how to build and prepare your Wasp app for deployment. You can then deploy the built Wasp app wherever and however you want, as long as your provider/server supports Wasp's build format.

After going through the general steps that apply to all deployments, you can follow step-by-step guides for deploying your Wasp app to the most popular providers:

-   [Fly.io](#flyio-server-and-database)
-   [Netlify](#netlify-client)
-   [Railway](#railway-server-client-and-database)
-   [Heroku](#heroku-server-and-database)

No worries, you can still deploy your app if your desired provider isn't on the list - it just means we don't yet have a step-by-step guide for you to follow. Feel free to [open a PR](https://github.com/wasp-lang/wasp/edit/release/web/docs/advanced/deployment/manually.md) if you'd like to write one yourself :)

## Deploying a Wasp App[​](#deploying-a-wasp-app "Direct link to Deploying a Wasp App")

Deploying a Wasp app comes down to the following:

1.  Generating deployable code.
2.  Deploying the API server (backend).
3.  Deploying the web client (frontend).
4.  Deploying a PostgreSQL database and keeping it running.

Let's go through each of these steps.

### 1\. Generating Deployable Code[​](#1-generating-deployable-code "Direct link to 1. Generating Deployable Code")

Running the command `wasp build` generates deployable code for the whole app in the `.wasp/build/` directory.

```
    wasp build
```

PostgreSQL in production

You won't be able to build the app if you are using SQLite as a database (which is the default database). You'll have to [switch to PostgreSQL](/docs/data-model/backends#migrating-from-sqlite-to-postgresql) before deploying to production.

### 2\. Deploying the API Server (backend)[​](#2-deploying-the-api-server-backend "Direct link to 2. Deploying the API Server (backend)")

There's a Dockerfile that defines an image for building the server in the `.wasp/build` directory.

To run the server in production, deploy this Docker image to a hosting provider and ensure the required environment variables on the provider are correctly set up (the mechanism of setting these up is specific per provider). All necessary environment variables are listed in the next section.

#### Environment Variables[​](#environment-variables "Direct link to Environment Variables")

Here are the environment variables your server will be looking for:

-   `DATABASE_URL` required
    
    The URL of the PostgreSQL database you want your app to use (e.g., `postgresql://mydbuser:mypass@localhost:5432/nameofmydb`).
    
-   `WASP_WEB_CLIENT_URL` required
    
    The URL where you plan to deploy your frontend app is running (e.g., `https://<app-name>.netlify.app`). The server needs to know about it to properly configure Same-Origin Policy (CORS) headers.
    
-   `WASP_SERVER_URL` required
    
    The URL where the server is running (e.g., `https://<app-name>.fly.dev`). The server needs it to properly redirect users when logging in with OAuth providers like Google or GitHub.
    
-   `JWT_SECRET` (required if using Wasp Auth)
    
    You only need this environment variable if you're using Wasp's `auth` features. Set it to a random string at least 32 characters long (you can use an [online generator](https://djecrety.ir/)).
    
-   `PORT`
    
    The server's HTTP port number. This is where the server listens for requests (default: `3001`).
    

Using an external auth method?

If your app is using an external authentication method(s) supported by Wasp (such as [Google](/docs/auth/social-auth/google#4-adding-environment-variables) or [GitHub](/docs/auth/social-auth/github#4-adding-environment-variables)), make sure to additionally set the necessary environment variables specifically required by these method(s).

While these are the general instructions on deploying the server anywhere, we also have more detailed instructions for chosen providers below, so check that out for more guidance if you are deploying to one of those providers.

### 3\. Deploying the Web Client (frontend)[​](#3-deploying-the-web-client-frontend "Direct link to 3. Deploying the Web Client (frontend)")

To build the web app, position yourself in `.wasp/build/web-app` directory:

```
    cd .wasp/build/web-app
```

Run

```
    npm install && REACT_APP_API_URL=<url_to_wasp_backend> npm run build
```

where `<url_to_wasp_backend>` is the URL of the Wasp server that you previously deployed.

Client Environment Variables

Remember, if you have manually defined any other [client-side environment variables](/docs/project/env-vars#client-env-vars) in your project, make sure to add them to the command above when [building your client](/docs/project/env-vars#client-env-vars-1)

The command above will build the web client and put it in the `build/` directory in the `.wasp/build/web-app/`.

This is also the moment to provide any additional env vars for the client code, next to `REACT_APP_API_URL`. Check the [env vars docs](/docs/project/env-vars#client-env-vars-1) for more details.

Since the result of building is just a bunch of static files, you can now deploy your web client to any static hosting provider (e.g. Netlify, Cloudflare, ...) by deploying the contents of `.wasp/build/web-app/build/`.

### 4\. Deploying the Database[​](#4-deploying-the-database "Direct link to 4. Deploying the Database")

Any PostgreSQL database will do, as long as you provide the server with the correct `DATABASE_URL` env var and ensure that the database is accessible from the server.

## Different Providers[​](#different-providers "Direct link to Different Providers")

We'll cover a few different deployment providers below:

-   Fly.io (server and database)
-   Netlify (client)
-   Railway (server, client and database)
-   Heroku (server and database)

## Fly.io (server and database)[​](#flyio-server-and-database "Direct link to Fly.io (server and database)")

We will show how to deploy the server and provision a database for it on Fly.io.

We automated this process for you

If you want to do all of the work below with one command, you can use the [Wasp CLI](/docs/advanced/deployment/cli#flyio).

Wasp CLI deploys the server, deploys the client, and sets up a database. It also gives you a way to redeploy (update) your app with a single command.

Fly.io offers a variety of free services that are perfect for deploying your first Wasp app! You will need a Fly.io account and the [`flyctl` CLI](https://fly.io/docs/hands-on/install-flyctl/).

note

Fly.io offers support for both locally built Docker containers and remotely built ones. However, for simplicity and reproducibility, we will default to the use of a remote Fly.io builder.

Additionally, `fly` is a symlink for `flyctl` on most systems and they can be used interchangeably.

Make sure you are logged in with `flyctl` CLI. You can check if you are logged in with `flyctl auth whoami`, and if you are not, you can log in with `flyctl auth login`.

### Set Up a Fly.io App[​](#set-up-a-flyio-app "Direct link to Set Up a Fly.io App")

info

You need to do this only once per Wasp app.

Unless you already have a Fly.io app that you want to deploy to, let's create a new Fly.io app.

After you have [built the app](#1-generating-deployable-code), position yourself in `.wasp/build/` directory:

```
    cd .wasp/build
```

Next, run the launch command to set up a new app and create a `fly.toml` file:

```
    flyctl launch --remote-only
```

This will ask you a series of questions, such as asking you to choose a region and whether you'd like a database.

-   Say **yes** to **Would you like to set up a PostgreSQL database now?** and select **Development**. Fly.io will set a `DATABASE_URL` for you.
    
-   Say **no** to **Would you like to deploy now?** (and to any additional questions).
    
    We still need to set up several environment variables.
    

What if the database setup fails?

If your attempts to initiate a new app fail for whatever reason, then you should run `flyctl apps destroy <app-name>` before trying again. Fly does not allow you to create multiple apps with the same name.

What does it look like when your DB is deployed correctly?

When your DB is deployed correctly, you'll see it in the [Fly.io dashboard](https://fly.io/dashboard):

![image](/img/deploying/fly-db.png)

Next, let's copy the `fly.toml` file up to our Wasp project dir for safekeeping.

```
    cp fly.toml ../../
```

Next, add a few more environment variables for the server code.

```
    flyctl secrets set PORT=8080flyctl secrets set JWT_SECRET=<random_string_at_least_32_characters_long>flyctl secrets set WASP_WEB_CLIENT_URL=<url_of_where_client_will_be_deployed>flyctl secrets set WASP_SERVER_URL=<url_of_where_server_will_be_deployed>
```

note

If you do not know what your client URL is yet, don't worry. You can set `WASP_WEB_CLIENT_URL` after you deploy your client.

Using an external auth method?

If your app is using an external authentication method(s) supported by Wasp (such as [Google](/docs/auth/social-auth/google#4-adding-environment-variables) or [GitHub](/docs/auth/social-auth/github#4-adding-environment-variables)), make sure to additionally set the necessary environment variables specifically required by these method(s).

If you want to make sure you've added your secrets correctly, run `flyctl secrets list` in the terminal. Note that you will see hashed versions of your secrets to protect your sensitive data.

### Deploy to a Fly.io App[​](#deploy-to-a-flyio-app "Direct link to Deploy to a Fly.io App")

While still in the `.wasp/build/` directory, run:

```
    flyctl deploy --remote-only --config ../../fly.toml
```

This will build and deploy the backend of your Wasp app on Fly.io to `https://<app-name>.fly.dev` 🤘🎸

Now, if you haven't, you can deploy your client and add the client URL by running `flyctl secrets set WASP_WEB_CLIENT_URL=<url_of_deployed_client>`. We suggest using [Netlify](#netlify) for your client, but you can use any static hosting provider.

Additionally, some useful `flyctl` commands:

```
    flyctl logsflyctl secrets listflyctl ssh console
```

### Redeploying After Wasp Builds[​](#redeploying-after-wasp-builds "Direct link to Redeploying After Wasp Builds")

When you rebuild your Wasp app (with `wasp build`), it will remove your `.wasp/build/` directory. In there, you may have a `fly.toml` from any prior Fly.io deployments.

While we will improve this process in the future, in the meantime, you have a few options:

1.  Copy the `fly.toml` file to a versioned directory, like your Wasp project dir.
    
    From there, you can reference it in `flyctl deploy --config <path>` commands, like above.
    
2.  Backup the `fly.toml` file somewhere before running `wasp build`, and copy it into .wasp/build/ after.
    
    When the `fly.toml` file exists in .wasp/build/ dir, you do not need to specify the `--config <path>`.
    
3.  Run `flyctl config save -a <app-name>` to regenerate the `fly.toml` file from the remote state stored in Fly.io.
    

## Netlify (client)[​](#netlify-client "Direct link to Netlify (client)")

We'll show how to deploy the client on Netlify.

Netlify is a static hosting solution that is free for many use cases. You will need a Netlify account and [Netlify CLI](https://docs.netlify.com/cli/get-started/) installed to follow these instructions.

Make sure you are logged in with Netlify CLI. You can check if you are logged in with `netlify status`, and if you are not, you can log in with `netlify login`.

First, make sure you have [built the Wasp app](#1-generating-deployable-code). We'll build the client web app next.

To build the web app, position yourself in `.wasp/build/web-app` directory:

```
    cd .wasp/build/web-app
```

Run

```
    npm install && REACT_APP_API_URL=<url_to_wasp_backend> npm run build
```

where `<url_to_wasp_backend>` is the URL of the Wasp server that you previously deployed.

Client Environment Variables

Remember, if you have manually defined any other [client-side environment variables](/docs/project/env-vars#client-env-vars) in your project, make sure to add them to the command above when [building your client](/docs/project/env-vars#client-env-vars-1)

We can now deploy the client with:

```
    netlify deploy
```

Carefully follow the instructions i.e. do you want to create a new app or use an existing one, the team under which your app will reside etc.

The final step is to run:

```
    netlify deploy --prod
```

That is it! Your client should be live at `https://<app-name>.netlify.app` ✨

note

Make sure you set this URL as the `WASP_WEB_CLIENT_URL` environment variable in your server hosting environment (e.g., Fly.io or Heroku).

## Railway (server, client and database)[​](#railway-server-client-and-database "Direct link to Railway (server, client and database)")

We will show how to deploy the client, the server, and provision a database on Railway.

Railway is a simple and great way to host your server and database. It's also possible to deploy your entire app: database, server, and client. You can use the platform for free for a limited time, or if you meet certain eligibility requirements. See their [plans page](https://docs.railway.app/reference/pricing/plans) for more info.

### Prerequisites[​](#prerequisites "Direct link to Prerequisites")

To get started, follow these steps:

1.  Make sure your Wasp app is built by running `wasp build` in the project dir.
    
2.  Create a [Railway](https://railway.app/) account
    
    Free Tier
    
    Sign up with your GitHub account to be eligible for the free tier
    
3.  Install the [Railway CLI](https://docs.railway.app/develop/cli#installation)
    
4.  Run `railway login` and a browser tab will open to authenticate you.
    

### Create New Project[​](#create-new-project "Direct link to Create New Project")

Let's create our Railway project:

1.  Go to your [Railway dashboard](https://railway.app/dashboard), click on **New Project**, and select `Provision PostgreSQL` from the dropdown menu.
2.  Once it initializes, right-click on the **New** button in the top right corner and select **Empty Service**.
3.  Once it initializes, click on it, go to **Settings > General** and change the name to `server`
4.  Go ahead and create another empty service and name it `client`

![Changing the name](/assets/images/railway-rename-78e07d8f6359fdb9eca0dc982357fb89.png)

### Deploy Your App to Railway[​](#deploy-your-app-to-railway "Direct link to Deploy Your App to Railway")

#### Setup Domains[​](#setup-domains "Direct link to Setup Domains")

We'll need the domains for both the `server` and `client` services:

1.  Go to the `server` instance's `Settings` tab, and click `Generate Domain`.
2.  Do the same under the `client`'s `Settings`.

Copy the domains as we will need them later.

#### Deploying the Server[​](#deploying-the-server "Direct link to Deploying the Server")

Let's deploy our server first:

1.  Move into your app's `.wasp/build/` directory:
    
    ```
        cd .wasp/build
    ```
    
2.  Link your app build to your newly created Railway project:
    
    ```
        railway link
    ```
    
3.  Go into the Railway dashboard and set up the required env variables:
    
    Open the `Settings` and go to the `Variables` tab:
    
    -   click **Variable reference** and select `DATABASE_URL` (it will populate it with the correct value)
        
    -   add `WASP_WEB_CLIENT_URL` - enter the `client` domain (e.g. `https://client-production-XXXX.up.railway.app`). `https://` prefix is required!
        
    -   add `WASP_SERVER_URL` - enter the `server` domain (e.g. `https://server-production-XXXX.up.railway.app`). `https://` prefix is required!
        
    -   add `JWT_SECRET` - enter a random string at least 32 characters long (use an [online generator](https://djecrety.ir/))
        
        Using an external auth method?
        
        If your app is using an external authentication method(s) supported by Wasp (such as [Google](/docs/auth/social-auth/google#4-adding-environment-variables) or [GitHub](/docs/auth/social-auth/github#4-adding-environment-variables)), make sure to additionally set the necessary environment variables specifically required by these method(s).
        
4.  Push and deploy the project:
    

```
    railway up
```

Select `server` when prompted with `Select Service`.

Railway will now locate the Dockerfile and deploy your server 👍

#### Deploying the Client[​](#deploying-the-client "Direct link to Deploying the Client")

1.  Next, change into your app's frontend build directory `.wasp/build/web-app`:
    
    ```
        cd web-app
    ```
    
2.  Create the production build, using the `server` domain as the `REACT_APP_API_URL`:
    
    ```
        npm install && REACT_APP_API_URL=<url_to_wasp_backend> npm run build
    ```
    
3.  Next, we want to link this specific frontend directory to our project as well:
    
    ```
        railway link
    ```
    
4.  We need to configure Railway's static hosting for our client.
    
    Setting Up Static Hosting
    
    Copy the `build` folder within the `web-app` directory to `dist`:
    
    ```
        cp -r build dist
    ```
    
    We'll need to create the following files:
    
    -   `Dockerfile` with:
        
        Dockerfile
        
        ```
            FROM pierrezemb/gostaticCMD [ "-fallback", "index.html" ]COPY ./dist/ /srv/http/
        ```
        
    -   `.dockerignore` with:
        
        .dockerignore
        
        ```
            node_modules/
        ```
        
    
    You'll need to repeat these steps **each time** you run `wasp build` as it will remove the `.wasp/build/web-app` directory.
    
    Here's a useful shell script to do the process
    
    If you want to automate the process, save the following as `deploy_client.sh` in the root of your project:
    
    deploy\_client.sh
    
    ```
        #!/usr/bin/env bashif [ -z "$REACT_APP_API_URL" ]then  echo "REACT_APP_API_URL is not set"  exit 1fiwasp buildcd .wasp/build/web-appnpm install && REACT_APP_API_URL=$REACT_APP_API_URL npm run buildcp -r build distdockerfile_contents=$(cat <<EOFFROM pierrezemb/gostaticCMD [ "-fallback", "index.html" ]COPY ./dist/ /srv/http/EOF)dockerignore_contents=$(cat <<EOFnode_modules/EOF)echo "$dockerfile_contents" > Dockerfileecho "$dockerignore_contents" > .dockerignorerailway up
    ```
    
    Make it executable with:
    
    ```
        chmod +x deploy_client.sh
    ```
    
    You can run it with:
    
    ```
        REACT_APP_API_URL=<url_to_wasp_backend> ./deploy_client.sh
    ```
    
5.  Set the `PORT` environment variable to `8043` under the `Variables` tab.
    
6.  Once set, deploy the client and select `client` when prompted with `Select Service`:
    

```
    railway up
```

#### Conclusion[​](#conclusion "Direct link to Conclusion")

And now your Wasp should be deployed! 🐝 🚂 🚀

Back in your [Railway dashboard](https://railway.app/dashboard), click on your project and you should see your newly deployed services: PostgreSQL, Server, and Client.

### Updates & Redeploying[​](#updates--redeploying "Direct link to Updates & Redeploying")

When you make updates and need to redeploy:

-   run `wasp build` to rebuild your app
-   run `railway up` in the `.wasp/build` directory (server)
-   repeat all the steps in the `.wasp/build/web-app` directory (client)

## Heroku (server and database)[​](#heroku-server-and-database "Direct link to Heroku (server and database)")

We will show how to deploy the server and provision a database for it on Heroku.

note

Heroku used to offer free apps under certain limits. However, as of November 28, 2022, they ended support for their free tier. [https://blog.heroku.com/next-chapter](https://blog.heroku.com/next-chapter)

As such, we recommend using an alternative provider like [Fly.io](#flyio) for your first apps.

You will need Heroku account, `heroku` [CLI](https://devcenter.heroku.com/articles/heroku-cli) and `docker` CLI installed to follow these instructions.

Make sure you are logged in with `heroku` CLI. You can check if you are logged in with `heroku whoami`, and if you are not, you can log in with `heroku login`.

### Set Up a Heroku App[​](#set-up-a-heroku-app "Direct link to Set Up a Heroku App")

info

You need to do this only once per Wasp app.

Unless you want to deploy to an existing Heroku app, let's create a new Heroku app:

```
    heroku create <app-name>
```

Unless you have an external PostgreSQL database that you want to use, let's create a new database on Heroku and attach it to our app:

```
    heroku addons:create --app <app-name> heroku-postgresql:mini
```

caution

Heroku does not offer a free plan anymore and `mini` is their cheapest database instance - it costs $5/mo.

Heroku will also set `DATABASE_URL` env var for us at this point. If you are using an external database, you will have to set it up yourself.

The `PORT` env var will also be provided by Heroku, so the ones left to set are the `JWT_SECRET`, `WASP_WEB_CLIENT_URL` and `WASP_SERVER_URL` env vars:

```
    heroku config:set --app <app-name> JWT_SECRET=<random_string_at_least_32_characters_long>heroku config:set --app <app-name> WASP_WEB_CLIENT_URL=<url_of_where_client_will_be_deployed>heroku config:set --app <app-name> WASP_SERVER_URL=<url_of_where_server_will_be_deployed>
```

note

If you do not know what your client URL is yet, don't worry. You can set `WASP_WEB_CLIENT_URL` after you deploy your client.

### Deploy to a Heroku App[​](#deploy-to-a-heroku-app "Direct link to Deploy to a Heroku App")

After you have [built the app](#1-generating-deployable-code), position yourself in `.wasp/build/` directory:

```
    cd .wasp/build
```

assuming you were at the root of your Wasp project at that moment.

Log in to Heroku Container Registry:

```
    heroku container:login
```

Build the docker image and push it to Heroku:

```
    heroku container:push --app <app-name> web
```

App is still not deployed at this point. This step might take some time, especially the very first time, since there are no cached docker layers.

Note for Apple Silicon Users

Apple Silicon users need to build a non-Arm image, so the above step will not work at this time. Instead of `heroku container:push`, users instead should:

```
    docker buildx build --platform linux/amd64 -t <app-name> .docker tag <app-name> registry.heroku.com/<app-name>/webdocker push registry.heroku.com/<app-name>/web
```

You are now ready to proceed to the next step.

Deploy the pushed image and restart the app:

```
    heroku container:release --app <app-name> web
```

This is it, the backend is deployed at `https://<app-name>-XXXX.herokuapp.com` 🎉

Find out the exact app URL with:

```
    heroku info --app <app-name>
```

Additionally, you can check out the logs with:

```
    heroku logs --tail --app <app-name>
```

Using `pg-boss` with Heroku

If you wish to deploy an app leveraging [Jobs](/docs/advanced/jobs) that use `pg-boss` as the executor to Heroku, you need to set an additional environment variable called `PG_BOSS_NEW_OPTIONS` to `{"connectionString":"<REGULAR_HEROKU_DATABASE_URL>","ssl":{"rejectUnauthorized":false}}`. This is because pg-boss uses the `pg` extension, which does not seem to connect to Heroku over SSL by default, which Heroku requires. Additionally, Heroku uses a self-signed cert, so we must handle that as well.

Read more: [https://devcenter.heroku.com/articles/connecting-heroku-postgres#connecting-in-node-js](https://devcenter.heroku.com/articles/connecting-heroku-postgres#connecting-in-node-js)

## Koyeb (server, client and database)[​](#koyeb-server-client-and-database "Direct link to Koyeb (server, client and database)")

Check out the tutorial made by the team at Koyeb for detailed instructions on how to deploy a whole Wasp app on Koyeb: [Using Wasp to Build Full-Stack Web Applications on Koyeb](https://www.koyeb.com/tutorials/using-wasp-to-build-full-stack-web-applications-on-koyeb).

The tutorial was written for Wasp v0.13.

---

# Wasp Language (.wasp)

Wasp language (what you write in .wasp files) is a declarative, statically typed, domain-specific language (DSL).

It is a quite simple language, closer to JSON, CSS or SQL than to e.g. Javascript or Python, since it is not a general programming language, but more of a configuration language.

It is pretty intuitive to learn (there isn't much to learn really!) and you can probably do just fine without reading this page and learning from the rest of the docs as you go, but if you want a bit more formal definition and deeper understanding of how it works, then read on!

## Declarations[​](#declarations "Direct link to Declarations")

The central point of Wasp language are **declarations**, and Wasp code is at the end just a bunch of declarations, each of them describing a part of your web app.

```
    app MyApp {  title: "My app"}route RootRoute { path: "/", to: DashboardPage }page DashboardPage {  component: import { DashboardPage } from "@src/Dashboard.jsx"}
```

In the example above we described a web app via three declarations: `app MyApp { ... }`, `route RootRoute { ... }` and `page DashboardPage { ... }`.

Syntax for writing a declaration is `<declaration_type> <declaration_name> <declaration_body>`, where:

-   `<declaration_type>` is one of the declaration types offered by Wasp (`app`, `route`, ...)
-   `<declaration_name>` is an identifier chosen by you to name this specific declaration
-   `<declaration_body>` is the value/definition of the declaration itself, which has to match the specific declaration body type expected by the chosen declaration type.

So, for `app` declaration above, we have:

-   declaration type `app`
-   declaration name `MyApp` (we could have used any other identifier, like `foobar`, `foo_bar`, or `hi3Ho`)
-   declaration body `{ title: "My app" }`, which is a dictionary with field `title` that has string value. Type of this dictionary is in line with the declaration body type of the `app` declaration type. If we provided something else, e.g. changed `title` to `little`, we would get a type error from Wasp compiler since that does not match the expected type of the declaration body for `app`.

Each declaration has a meaning behind it that describes how your web app should behave and function.

All the other types in Wasp language (primitive types (`string`, `number`), composite types (`dict`, `list`), enum types (`DbSystem`), ...) are used to define the declaration bodies.

## Complete List of Wasp Types[​](#complete-list-of-wasp-types "Direct link to Complete List of Wasp Types")

Wasp's type system can be divided into two main categories of types: **fundamental types** and **domain types**.

While fundamental types are here to be basic building blocks of a language and are very similar to what you would see in other popular languages, domain types are what make Wasp special, as they model the concepts of a web app like `page`, `route` and similar.

-   Fundamental types ([source of truth](https://github.com/wasp-lang/wasp/blob/main/waspc/src/Wasp/Analyzer/Type.hs))
    -   Primitive types
        -   **string** (`"foo"`, `"they said: \"hi\""`)
        -   **bool** (`true`, `false`)
        -   **number** (`12`, `14.5`)
        -   **declaration reference** (name of existing declaration: `TaskPage`, `updateTask`)
        -   **ExtImport** (external import) (`import Foo from "@src/bar.js"`, `import { Smth } from "@src/a/b.js"`)
            -   The path has to start with "@src". The rest is relative to the `src` directory.
            -   Import has to be a default import `import Foo` or a single named import `import { Foo }`.
        -   **json** (`{=json { a: 5, b: ["hi"] } json=}`)
    -   Composite types
        -   **dict** (dictionary) (`{ a: 5, b: "foo" }`)
        -   **list** (`[1, 2, 3]`)
        -   **tuple** (`(1, "bar")`, `(2, 4, true)`)
            -   Tuples can be of size 2, 3 and 4.
-   Domain types ([source of truth](https://github.com/wasp-lang/wasp/blob/main/waspc/src/Wasp/Analyzer/StdTypeDefinitions.hs))
    -   Declaration types
        -   **action**
        -   **api**
        -   **apiNamespace**
        -   **app**
        -   **job**
        -   **page**
        -   **query**
        -   **route**
        -   **crud**
    -   Enum types
        -   **DbSystem**
        -   **HttpMethod**
        -   **JobExecutor**
        -   **EmailProvider**
    -   Models from the `schema.prisma` file
        -   You can reference models defined in the `schema.prisma` file in your Wasp file by using the model name e.g. `Task`.

You can find more details about each of the domain types, both regarding their body types and what they mean, in the corresponding doc pages covering their features.

---

# CLI Reference

This guide provides an overview of the Wasp CLI commands, arguments, and options.

## Overview[​](#overview "Direct link to Overview")

Once [installed](/docs/quick-start), you can use the wasp command from your command line.

If you run the `wasp` command without any arguments, it will show you a list of available commands and their descriptions:

```
    USAGE  wasp <command> [command-args]COMMANDS  GENERAL    new [<name>] [args]   Creates a new Wasp project. Run it without arguments for interactive mode.      OPTIONS:        -t|--template <template-name>           Check out the templates list here: https://github.com/wasp-lang/starters    new:ai <app-name> <app-description> [<config-json>]      Uses AI to create a new Wasp project just based on the app name and the description.      You can do the same thing with `wasp new` interactively.      Run `wasp new:ai` for more info.    version               Prints current version of CLI.    waspls                Run Wasp Language Server. Add --help to get more info.    completion            Prints help on bash completion.    uninstall             Removes Wasp from your system.  IN PROJECT    start                 Runs Wasp app in development mode, watching for file changes.    start db              Starts managed development database for you.    db <db-cmd> [args]    Executes a database command. Run 'wasp db' for more info.    clean                 Deletes all generated code, all cached artifacts, and the node_modules dir.                          Wasp equivalent of 'have you tried closing and opening it again?'.    build                 Generates full web app code, ready for deployment. Use when deploying or ejecting.    deploy                Deploys your Wasp app to cloud hosting providers.    telemetry             Prints telemetry status.    deps                  Prints the dependencies that Wasp uses in your project.    dockerfile            Prints the contents of the Wasp generated Dockerfile.    info                  Prints basic information about the current Wasp project.    test                  Executes tests in your project.    studio                (experimental) GUI for inspecting your Wasp app.EXAMPLES  wasp new MyApp  wasp start  wasp db migrate-devDocs: https://wasp-lang.dev/docsDiscord (chat): https://discord.gg/rzdnErXNewsletter: https://wasp-lang.dev/#signup
```

## Commands[​](#commands "Direct link to Commands")

### Creating a New Project[​](#creating-a-new-project "Direct link to Creating a New Project")

-   Use `wasp new` to start the interactive mode for setting up a new Wasp project.
    
    This will prompt you to input the project name and to select a template. The chosen template will then be used to generate the project directory with the specified name.
    
    ```
        $ wasp newEnter the project name (e.g. my-project) ▸ MyFirstProjectChoose a starter template[1] basic (default)    Simple starter template with a single page.[2] todo-ts    Simple but well-rounded Wasp app implemented with Typescript & full-stack type safety.[3] saas    Everything a SaaS needs! Comes with Auth, ChatGPT API, Tailwind, Stripe payments and more. Check out https://opensaas.sh/ for more details.[4] embeddings    Comes with code for generating vector embeddings and performing vector similarity search.[5] ai-generated    🤖 Describe an app in a couple of sentences and have Wasp AI generate initial code for you. (experimental)▸ 1🐝 --- Creating your project from the "basic" template... -------------------------Created new Wasp app in ./MyFirstProject directory!To run your new app, do:    cd MyFirstProject    wasp db start
    ```
    
-   To skip the interactive mode and create a new Wasp project with the default template, use `wasp new <project-name>`.
    
    ```
        $ wasp new MyFirstProject🐝 --- Creating your project from the "basic" template... -------------------------Created new Wasp app in ./MyFirstProject directory!To run your new app, do:   cd MyFirstProject   wasp db start
    ```
    

### Project Commands[​](#project-commands "Direct link to Project Commands")

-   `wasp start` launches the Wasp app in development mode. It automatically opens a browser tab with your application running and watches for any changes to .wasp or files in `src/` to automatically reflect in the browser. It also shows messages from the web app, the server and the database on stdout/stderr.
    
-   `wasp start db` starts the database for you. This can be very handy since you don't need to spin up your own database or provide its connection URL to the Wasp app.
    
-   `wasp clean` removes all generated code and other cached artifacts. If using SQlite, it also deletes the SQlite database. Think of this as the Wasp version of the classic "turn it off and on again" solution.
    
    ```
        $ wasp clean🐝 --- Deleting the .wasp/ directory... -------------------------------------------✅ --- Deleted the .wasp/ directory. ----------------------------------------------🐝 --- Deleting the node_modules/ directory... ------------------------------------✅ --- Deleted the node_modules/ directory. ---------------------------------------
    ```
    
-   `wasp build` generates the complete web app code, which is ready for [deployment](/docs/advanced/deployment/overview). Use this command when you're deploying or ejecting. The generated code is stored in the `.wasp/build` folder.
    
-   `wasp deploy` makes it easy to get your app hosted on the web.
    
    Currently, Wasp offers support for [Fly.io](https://fly.io). If you prefer a different hosting provider, feel free to let us know on Discord or submit a PR by updating [this TypeScript app](https://github.com/wasp-lang/wasp/tree/main/waspc/packages/deploy).
    
    Read more about automatic deployment [here](/docs/advanced/deployment/cli).
    
-   `wasp telemetry` displays the status of [telemetry](https://wasp-lang.dev/docs/telemetry).
    
    ```
        $ wasp telemetryTelemetry is currently: ENABLEDTelemetry cache directory: /home/user/.cache/wasp/telemetry/Last time telemetry data was sent for this project: 2021-05-27 09:21:16.79537226 UTCOur telemetry is anonymized and very limited in its scope: check https://wasp-lang.dev/docs/telemetry for more details.
    ```
    
-   `wasp deps` lists the dependencies that Wasp uses in your project.
    
-   `wasp info` provides basic details about the current Wasp project.
    
-   `wasp studio` shows you an graphical overview of your application in a graph: pages, queries, actions, data model etc.
    

### Database Commands[​](#database-commands "Direct link to Database Commands")

Wasp provides a suite of commands for managing the database. These commands all begin with `db` and primarily execute Prisma commands behind the scenes.

-   `wasp db migrate-dev` synchronizes the development database with the current state of the schema (entities). If there are any changes in the schema, it generates a new migration and applies any pending migrations to the database.
    
    -   The `--name foo` option allows you to specify a name for the migration, while the `--create-only` option lets you create an empty migration without applying it.
-   `wasp db studio` opens the GUI for inspecting your database.
    

using `prisma` CLI directly

Although Wasp uses the `schema.prisma` file to define the database schema, you must not use the `prisma` command directly. Instead, use the `wasp db` commands.

Wasp adds some additional functionality on top of Prisma, and using `prisma` commands directly can lead to unexpected behavior e.g. missing auth models, incorrect database setup, etc.

### Bash Completion[​](#bash-completion "Direct link to Bash Completion")

To set up Bash completion, run the `wasp completion` command and follow the instructions.

### Miscellaneous Commands[​](#miscellaneous-commands "Direct link to Miscellaneous Commands")

-   `wasp version` displays the current version of the CLI.
    
    ```
        $ wasp version0.14.0If you wish to install/switch to the latest version of Wasp, do:curl -sSL https://get.wasp-lang.dev/installer.sh | sh -sIf you want specific x.y.z version of Wasp, do:curl -sSL https://get.wasp-lang.dev/installer.sh | sh -s -- -v x.y.zCheck https://github.com/wasp-lang/wasp/releases for the list of valid versions, including the latest one.
    ```
    
-   `wasp uninstall` removes Wasp from your system.
    
    ```
        $ wasp uninstall🐝 --- Uninstalling Wasp ... ------------------------------------------------------ We will remove the following directories:   {home}/.local/share/wasp-lang/   {home}/.cache/wasp/ We will also remove the following files:   {home}/.local/bin/wasp Are you sure you want to continue? [y/N] y ✅ --- Uninstalled Wasp -----------------------------------------------------------
    ```

---

# TypeScript support

TypeScript is a programming language that adds static type analysis to JavaScript. It is a superset of JavaScript, which means all JavaScript code is valid TypeScript code. It also compiles to JavaScript before running.

TypeScript's type system helps catch errors at build time (this reduces runtime errors), and provides type-based auto-completion in IDEs.

Each Wasp feature includes TypeScript documentation.

If you're starting a new project and want to use TypeScript, you don't need to do anything special. Just follow the feature docs you are interested in, and they will tell you everything you need to know. We recommend you start by going through [the tutorial](/docs/tutorial/create).

To migrate an existing Wasp project from JavaScript to TypeScript, follow this guide.

## Migrating your project to TypeScript[​](#migrating-your-project-to-typescript "Direct link to Migrating your project to TypeScript")

Since Wasp ships with out-of-the-box TypeScript support, migrating your project is as simple as changing file extensions and using the language. This approach allows you to gradually migrate your project on a file-by-file basis.

We will first show you how to migrate a single file and then help you generalize the procedure to the rest of your project.

### Migrating a single file[​](#migrating-a-single-file "Direct link to Migrating a single file")

Assuming your `schema.prisma` file defines the `Task` entity:

schema.prisma

```
    // ...model Task {  id          Int @id @default(autoincrement())  description String  isDone      Boolean}
```

And your `main.wasp` file defines the `getTaskInfo` query:

main.wasp

```
    query getTaskInfo {  fn: import { getTaskInfo } from "@src/queries",  entities: [Task]}
```

We will show you how to migrate the following `queries.js` file:

src/queries.js

```
    import HttpError from 'wasp/server'function getInfoMessage(task) {  const isDoneText = task.isDone ? 'is done' : 'is not done'  return `Task '${task.description}' is ${isDoneText}.`}export const getTaskInfo = async ({ id }, context) => {  const Task = context.entities.Task  const task = await Task.findUnique({ where: { id } })  if (!task) {    throw new HttpError(404)  }  return getInfoMessage(task)}
```

To migrate this file to TypeScript, all you have to do is:

1.  Change the filename from `queries.js` to `queries.ts`.
2.  Write some types (and optionally use some of Wasp's TypeScript features).

-   Before
-   After

src/queries.js

```
    import HttpError from '@wasp/core/HttpError.js'function getInfoMessage(task) {  const isDoneText = task.isDone ? 'is done' : 'is not done'  return `Task '${task.description}' is ${isDoneText}.`}export const getTaskInfo = async ({ id }, context) => {  const Task = context.entities.Task  const task = await Task.findUnique({ where: { id } })  if (!task) {    throw new HttpError(404)  }  return getInfoMessage(task)}
```

src/queries.ts

```
    import HttpError from 'wasp/server'import { type Task } from '@wasp/entities'import { type GetTaskInfo } from '@wasp/server/operations'function getInfoMessage(task: Pick<Task, 'isDone' | 'description'>): string {  const isDoneText = task.isDone ? 'is done' : 'is not done'  return `Task '${task.description}' is ${isDoneText}.`}export const getTaskInfo: GetTaskInfo<Pick<Task, 'id'>, string> = async (  { id },  context) => {  const Task = context.entities.Task  const task = await Task.findUnique({ where: { id } })  if (!task) {    throw new HttpError(404)  }  return getInfoMessage(task)}
```

Your code is now processed by TypeScript and uses several of Wasp's TypeScript-specific features:

-   `Task` - A type that represents the `Task` entity. Using this type connects your data to the model definitions in the `schema.prisma` file. Read more about this feature [here](/docs/data-model/entities).
    
-   `GetTaskInfo<...>` - A generic type Wasp automatically generates to give you type support when implementing the Query. Thanks to this type, the compiler knows:
    
    -   The type of the `context` object.
    -   The type of `args`.
    -   The Query's return type.
    
    And gives you Intellisense and type-checking. Read more about this feature [here](/docs/data-model/operations/queries#implementing-queries).
    

You don't need to change anything inside the `.wasp` file.

### Migrating the rest of the project[​](#migrating-the-rest-of-the-project "Direct link to Migrating the rest of the project")

You can migrate your project gradually - on a file-by-file basis.

When you want to migrate a file, follow the procedure outlined above:

1.  Change the file's extension.
2.  Fix the type errors.
3.  Read the Wasp docs and decide which TypeScript features you want to use.

LSP Problems

If you are using TypeScript, your editor may sometimes report type and import errors even while `wasp start` is running.

This happens when the TypeScript Language Server gets out of sync with the current code. If you're using VS Code, you can manually restart the language server by opening the command palette and selecting *"TypeScript: Restart TS Server."* Open the command pallete with:

-   `Ctrl` + `Shift` + `P` if you're on Windows or Linux.
-   `Cmd` + `Shift` + `P` if you're on a Mac.

---

# Contributing

Any way you want to contribute is a good way, and we'd be happy to meet you! A single entry point for all contributors is the [CONTRIBUTING.md](https://github.com/wasp-lang/wasp/blob/main/CONTRIBUTING.md) file in our Github repo. All the requirements and instructions are there, so please check [CONTRIBUTING.md](https://github.com/wasp-lang/wasp/blob/main/CONTRIBUTING.md) for more details.

Some side notes to make your journey easier:

1.  Join us on [Discord](https://discord.gg/rzdnErX) and let's talk! We can discuss language design, new/existing features, and weather, or you can tell us how you feel about Wasp :).
    
2.  Wasp's compiler is built with Haskell. That means you'll need to be somewhat familiar with this language if you'd like to contribute to the compiler itself. But Haskell is just a part of Wasp, and you can contribute to lot of parts that require web dev skills, either by coding or by suggesting how to improve Wasp and its design as a web framework. If you don't have Haskell knowledge (or any dev experience at all) - no problem. There are a lot of JS-related tasks and documentation updates as well!
    
3.  If there's something you'd like to bring to our attention, go to [docs GitHub repo](https://github.com/wasp-lang/wasp) and make an issue/PR!
    

Happy hacking!

---

# Telemetry

## Overview[​](#overview "Direct link to Overview")

The term **telemetry** refers to the collection of certain usage data to help improve the quality of a piece of software (in this case, Wasp).

Our telemetry implementation is anonymized and very limited in its scope, focused on answering following questions:

-   How many people and how often: tried to install Wasp, use Wasp, have built a Wasp app, or have deployed one?
-   How many projects are created with Wasp?

## When and what is sent?[​](#when-and-what-is-sent "Direct link to When and what is sent?")

-   Information is sent via HTTPS request when `wasp` CLI command is invoked. Information is sent no more than twice in a period of 12 hours (sending is paused for 12 hours after last invocation, separately for `wasp build` command and for all other commands). Exact information as it is sent:
    
    ```
        {  // Randomly generated, non-identifiable UUID representing a user.  "distinct_id": "bf3fa7a8-1c11-4f82-9542-ec1a2d28786b",  // Non-identifiable hash representing a project.  "project_hash": "6d7e561d62b955d1",  // True if command was `wasp build`, false otherwise.  "is_build": true,  // Captures `wasp deploy ...` args, but only those from the limited, pre-defined list of keywords.  // Those are "fly", "setup", "create-db", "deploy" and "cmd". Everything else is ommited.  "deploy_cmd_args": "fly;deploy",  "wasp_version": "0.1.9.1",  "os": "linux",  // "CI" if running on CI, and whatever is the content of "WASP_TELEMETRY_CONTEXT" env var.  // We use this to track when execution is happening in some special context, like on Gitpod, Replit or similar.  "context": "CI"}
    ```
    
-   Information is also sent once via HTTPS request when wasp is installed via `install.sh` script. Exact information as it is sent:
    
    ```
        {  // Randomly generated id.  "distinct_id": "274701613078193779564259",  "os": "linux"}
    ```
    

## Opting out[​](#opting-out "Direct link to Opting out")

You sharing the telemetry data with us means a lot to us, since it helps us understand how popular Wasp is, how it is being used, how the changes we are doing affect usage, how many new vs old users there are, and just in general how Wasp is doing. We look at these numbers every morning and they drive us to make Wasp better.

However, if you wish to opt-out of telemetry, we understand! You can do so by setting the `WASP_TELEMETRY_DISABLE` environment variable to any value, e.g.:

```
    export WASP_TELEMETRY_DISABLE=1
```

## Future plans[​](#future-plans "Direct link to Future plans")

We don't have this implemented yet, but the next step will be to make telemetry go in two directions -> instead of just sending usage data to us, it will also at the same time check for any messages from our side (e.g. notification about new version of Wasp, or a security notice). [Link to corresponding github issue](https://github.com/wasp-lang/wasp/issues/163).

---

# Vision

With Wasp, we want to make developing web apps easy and enjoyable, for novices and experts in web development alike.

Ideal we are striving for is that programming in Wasp feels like describing an app using a human language - like writing a specification document where you describe primarily your requirements and as little implementation details as possible. Creating a new production-ready web app should be easy and deploying it to production should be straightforward.

That is why we believe Wasp needs to be a programming language (DSL) and not a library - we want to capture all parts of the web app into one integrated system that is perfectly tailored just for that purpose.  
On the other hand, we believe that trying to capture every single detail in one language would not be reasonable. There are solutions out there that work very well for the specific task they aim to solve (React for web components, CSS/HTML for design/markup, JS/TS for logic, ...) and we don't want to replace them with Wasp. Instead, we see Wasp as a declarative "glue" code uniting all these specific solutions and providing a higher-level notion of the web app above them.

Wasp is still early in its development and therefore far from where we imagine it will be in the future. This is what we imagine:

-   **Declarative, static language** with simple basic rules and **that understands a lot of web app concepts** - "horizontal language". Supports multiple files/modules, libraries.
-   **Integrates seamlessly with the most popular technologies** for building specific, more complex parts of the web app (React, CSS, JS, ...). They can be used inline (mixed with Wasp code) or provided via external files.
-   **Has hatches (escape mechanisms) that allow you to customize your web app** in all the right places, but remain hidden until you need them.
-   **Entity (data model) is a first-class citizen** - defined via custom Wasp syntax and it integrates very closely with the rest of the features, serving as one of the central concepts around which everything is built.
-   **Out of the box** support for CRUD UI based on the Entities, to get you quickly going, but also customizable to some level.
-   **"Smart" operations (queries and actions)** that in most cases automatically figure out when to update, and if not it is easy to define custom logic to compensate for that. User worries about client-server gap as little as possible.
-   Support, directly in Wasp, for **declaratively defining simple components and operations**.
-   Besides Wasp as a programming language, there will also be a **visual builder that generates/edits Wasp code**, allowing non-developers to participate in development. Since Wasp is declarative, we imagine such builder to naturally follow from Wasp language.
-   **Server side rendering, caching, packaging, security**, ... -> all those are taken care of by Wasp. You tell Wasp what you want, and Wasp figures out how to do it.
-   As **simple deployment to production/staging** as it gets.
-   While it comes with the official implementation(s), **Wasp language will not be coupled with the single implementation**. Others can provide implementations that compile to different web app stacks.

---

# Contact

You can find us on [Discord](https://discord.gg/rzdnErX) or you can reach out to us via email at [hi@wasp-lang.dev](mailto:hi@wasp-lang.dev).

---

# Migration from 0.11.X to 0.12.X

Migrating to the latest version

To fully migrate from 0.11.X to the latest version of Wasp, you should first migrate to **0.12.X** and then continue going through the migration guides.

Make sure to read the [migration guide from 0.12.X to 0.13.X](/docs/migrate-from-0-12-to-0-13) after you finish this one.

## What's new in Wasp 0.12.0?[​](#whats-new-in-wasp-0120 "Direct link to What's new in Wasp 0.12.0?")

### New project structure[​](#new-project-structure "Direct link to New project structure")

Here's a file tree of a fresh Wasp project created with the previous version of Wasp. More precisely, this is what you'll get if you run `wasp new myProject` using Wasp 0.11.x:

```
    .├── .gitignore├── main.wasp├── src│   ├── client│   │   ├── Main.css│   │   ├── MainPage.jsx│   │   ├── react-app-env.d.ts│   │   ├── tsconfig.json│   │   └── waspLogo.png│   ├── server│   │   └── tsconfig.json│   ├── shared│   │   └── tsconfig.json│   └── .waspignore└── .wasproot
```

Compare that with the file tree of a fresh Wasp project created with Wasp 0.12.0. In other words, this is what you will get by running `wasp new myProject` from this point onwards:

```
    .├── .gitignore├── main.wasp├── package.json├── public│   └── .gitkeep├── src│   ├── Main.css│   ├── MainPage.jsx│   ├── queries.ts│   ├── vite-env.d.ts│   ├── .waspignore│   └── waspLogo.png├── tsconfig.json├── vite.config.ts└── .wasproot
```

The main differences are:

-   The server/client code separation is no longer necessary. You can now organize your code however you want, as long as it's inside the `src` directory.
-   All external imports in your Wasp file must have paths starting with `@src` (e.g., `import foo from '@src/bar.js'`) where `@src` refers to the `src` directory in your project root. The paths can no longer start with `@server` or `@client`.
-   Your project now features a top-level `public` dir. Wasp will publicly serve all the files it finds in this directory. Read more about it [here](/docs/project/static-assets).

Our [Overview docs](/docs/tutorial/project-structure) explain the new structure in detail, while this page provides a [quick guide](#migrating-your-project-to-the-new-structure) for migrating existing projects.

### New auth[​](#new-auth "Direct link to New auth")

In Wasp 0.11.X, authentication was based on the `User` model which the developer needed to set up properly and take care of the auth fields like `email` or `password`.

main.wasp

```
    app myApp {  wasp: {    version: "^0.11.0"  },  title: "My App",  auth: {    userEntity: User,    externalAuthEntity: SocialLogin,    methods: {      gitHub: {}    },    onAuthFailedRedirectTo: "/login"  },}entity User {=psl  id                        Int           @id @default(autoincrement())  username                  String        @unique  password                  String  externalAuthAssociations  SocialLogin[]psl=}entity SocialLogin {=psl  id          Int       @id @default(autoincrement())  provider    String  providerId  String  user        User      @relation(fields: [userId], references: [id], onDelete: Cascade)  userId      Int  createdAt   DateTime  @default(now())  @@unique([provider, providerId, userId])psl=}
```

From 0.12.X onwards, authentication is based on the auth models which are automatically set up by Wasp. You don't need to take care of the auth fields anymore.

The `User` model is now just a business logic model and you use it for storing the data that is relevant for your app.

main.wasp

```
    app myApp {  wasp: {    version: "^0.12.0"  },  title: "My App",  auth: {    userEntity: User,    methods: {      gitHub: {}    },    onAuthFailedRedirectTo: "/login"  },}entity User {=psl  id Int @id @default(autoincrement())psl=}
```

Regression Note: Multiple Auth Identities per User

With our old auth implementation, if you were using both Google and email auth methods, your users could sign up with Google first and then, later on, reset their password and therefore also enable logging in with their email and password. This was the only way in which a single user could have multiple login methods at the same time (Google and email).

This is not possible anymore. **The new auth system doesn't support multiple login methods per user at the moment**. We do plan to add this soon though, with the introduction of the [account merging feature](https://github.com/wasp-lang/wasp/issues/954).

If you have any users that have both Google and email login credentials at the same time, you will have to pick only one of those for that user to keep when migrating them.

Regression Note: `_waspCustomValidations` is deprecated

Auth field customization is no longer possible using the `_waspCustomValidations` on the `User` entity. This is a part of auth refactoring that we are doing to make it easier to customize auth. We will be adding more customization options in the future.

You can read more about the new auth system in the [Accessing User Data](/docs/auth/entities) section.

## How to Migrate?[​](#how-to-migrate "Direct link to How to Migrate?")

These instructions are for migrating your app from Wasp `0.11.X` to Wasp `0.12.X`, meaning they will work for all minor releases that fit this pattern (e.g., the guide applies to `0.12.0`, `0.12.1`, ...).

The guide consists of two big steps:

1.  Migrating your Wasp project to the new structure.
2.  Migrating to the new auth.

If you get stuck at any point, don't hesitate to ask for help on [our Discord server](https://discord.gg/rzdnErX).

### Migrating Your Project to the New Structure[​](#migrating-your-project-to-the-new-structure "Direct link to Migrating Your Project to the New Structure")

You can easily migrate your old Wasp project to the new structure by following a series of steps. Assuming you have a project called `foo` inside the directory `foo`, you should:

0.  **Install the `0.12.x` version** of Wasp.
    
    ```
        curl -sSL https://get.wasp-lang.dev/installer.sh | sh -s -- -v 0.12.4
    ```
    
1.  Make sure to **backup or save your project** before starting the procedure (e.g., by committing it to source control or creating a copy).
2.  **Position yourself in the terminal** in the directory that is a parent of your wasp project directory (so one level above: if you do `ls`, you should see your wasp project dir listed).
3.  **Run the migration script** (replace `foo` at the end with the name of your Wasp project directory) and follow the instructions:
    
    ```
        npx wasp-migrate foo
    ```
    

In case the migration script doesn't work well for you, you can do the same steps manually, as described here:

1.  Rename your project's root directory to something like `foo_old`.
    
2.  Create a new project by running `wasp new foo`.
    
3.  Delete all files of `foo/src` except `vite-env.d.ts`.
    
4.  If `foo_old/src/client/public` exists and contains any files, copy those files into `foo/public`.
    
5.  Copy the contents of `foo_old/src` into `foo/src`. `foo/src` should now contain `vite-env.d.ts`, `.waspignore`, and three subdirectories (`server`, `client`, and `shared`). Don't change anything about this structure yet.
    
6.  Delete redundant files and folders from `foo/src`:
    
    -   `foo/src/.waspignore` - A new version of this file already exists at the top level.
    -   `foo/src/client/vite-env.d.ts` - A new version of this file already exists at the top level.
    -   `foo/src/client/tsconfig.json` - A new version of this file already exists at the top level.
    -   `foo/src/server/tsconfig.json` - A new version of this file already exists at the top level.
    -   `foo/src/shared/tsconfig.json` - A new version of this file already exists at the top level.
    -   `foo/src/client/public` - You've moved all the files from this directory in step 5.
7.  Update all the `@wasp` imports in your JS(X)/TS(X) source files in the `src/` dir.
    
    For this, we prepared a special script that will rewrite these imports automatically for you.
    
    Before doing this step, as the script will modify your JS(X)/TS(X) files in place, we advise committing all changes you have so far, so you can then both easily inspect the import rewrites that our script did (with `git diff`) and also revert them if something went wrong.
    
    To run the import-rewriting script, make sure you are in the root dir of your wasp project, and then run
    
    ```
        npx jscodeshift@0.15.1 -t https://raw.githubusercontent.com/wasp-lang/wasp-codemod/main/src/transforms/imports-from-0-11-to-0-12.ts --extensions=js,ts,jsx,tsx src/
    ```
    
    Then, check the changes it did, in case some kind of manual intervention is needed (in which case you should see TODO comments generated by the script).
    
    Alternatively, you can find all the mappings of old imports to the new ones in [this table](https://docs.google.com/spreadsheets/d/1QW-_16KRGTOaKXx9NYUtjk6m2TQ0nUMOA74hBthTH3g/edit#gid=1725669920) and use it to fix some/all of them manually.
    
8.  Replace the Wasp file in `foo` (i.e., `main.wasp`) with the Wasp file from `foo_old`
    
9.  Change the Wasp version field in your Wasp file (now residing in `foo`) to `"^0.12.0"`.
    
10.  Correct external imports in your Wasp file (now residing in `foo`). imports. You can do this by running search-and-replace inside the file:
    
    -   Change all occurrences of `@server` to `@src/server`
    -   Change all occurrences of `@client` to `@src/client`
    
    For example, if you previously had something like:
    
    ```
        page LoginPage {  // This previously resolved to src/client/LoginPage.js  component: import Login from "@client/LoginPage"}// ...query getTasks {  // This previously resolved to src/server/queries.js  fn: import { getTasks } from "@server/queries.js",}
    ```
    
    You should change it to:
    
    ```
        page LoginPage {  // This now resolves to src/client/LoginPage.js  component: import Login from "@src/client/LoginPage"}// ...query getTasks {  // This now resolves to src/server/queries.js  fn: import { getTasks } from "@src/server/queries.js",}
    ```
    
    Do this for all external imports in your `.wasp` file. After you're done, there shouldn't be any occurrences of strings `"@server"` or `"@client"`
    
11.  Take all the dependencies from `app.dependencies` declaration in `foo/main.wasp` and move them to `foo/package.json`. Make sure to remove the `app.dependencies` field from `foo/main.wasp`.
    
    For example, if `foo_old/main.wasp` had:
    
    ```
        app Foo {  // ...  dependencies: [ ('redux', '^4.0.5'), ('reacjt-redux', '^7.1.3')];}
    ```
    
    Your `package.json` in `foo` should now list these dependencies (Wasp already generated most of the file, you just have to list additional dependencies).
    
    ```
        {  "name": "foo",  "dependencies": {    "wasp": "file:.wasp/out/sdk/wasp",    "react": "^18.2.0",    "redux": "^4.0.5",    "reactjs-redux": "^7.1.3"  },  "devDependencies": {    "typescript": "^5.1.0",    "vite": "^4.3.9",    "@types/react": "^18.0.37",    "prisma": "4.16.2"  }}
    ```
    
12.  Copy all lines you might have added to `foo_old/.gitignore` into `foo/.gitignore`
    
13.  Copy the rest of the top-level files and folders (all of them except for `.gitignore`, `main.wasp` and `src/`) in `foo_old/` into `foo/` (overwrite the existing files in `foo`).
    
14.  Run `wasp clean` in `foo`.
    
15.  Delete the `foo_old` directory.
    

That's it! You now have a properly structured Wasp 0.12.0 project in the `foo` directory. Your app probably doesn't quite work yet due to some other changes in Wasp 0.12.0, but we'll get to that in the next sections.

### Migrating declaration names[​](#migrating-declaration-names "Direct link to Migrating declaration names")

Wasp 0.12.0 adds a casing constraints when naming Queries, Actions, Jobs, and Entities in the `main.wasp` file.

The following casing conventions have now become mandatory:

-   Operation (i.e., Query and Action) names must begin with a lowercase letter: `query getTasks {...}`, `action createTask {...}`.
-   Job names must begin with a lowercase letter: `job sendReport {...}`.
-   Entity names must start with an uppercase letter: `entity Task {...}`.

### Migrating the Tailwind Setup[​](#migrating-the-tailwind-setup "Direct link to Migrating the Tailwind Setup")

note

If you don't use Tailwind in your project, you can skip this section.

There is a small change in how the `tailwind.config.cjs` needs to be defined in Wasp 0.12.0.

You'll need to wrap all your paths in the `content` field with the `resolveProjectPath` function. This makes sure that the paths are resolved correctly when generating your CSS.

Here's how you can do it:

-   Before
-   After

tailwind.config.cjs

```
    /** @type {import('tailwindcss').Config} */module.exports = {  content: [    './src/**/*.{js,jsx,ts,tsx}',  ],  theme: {    extend: {},  },  plugins: [],}
```

tailwind.config.cjs

```
    const { resolveProjectPath } = require('wasp/dev')/** @type {import('tailwindcss').Config} */module.exports = {  content: [    resolveProjectPath('./src/**/*.{js,jsx,ts,tsx}'),  ],  theme: {    extend: {},  },  plugins: [],}
```

### Default Server Dockerfile Changed[​](#default-server-dockerfile-changed "Direct link to Default Server Dockerfile Changed")

note

If you didn't customize your Dockerfile or had a custom build process for the Wasp server, you can skip this section.

Between Wasp 0.11.X and 0.12.X, the Dockerfile that Wasp generates for you for deploying the server has changed. If you defined a custom Dockerfile in your project root dir or in any other way relied on its contents, you'll need to update it to incorporate the changes that Wasp 0.12.X made.

We suggest that you temporarily move your custom Dockerfile to a different location, then run `wasp start` to generate the new Dockerfile. Check out the `.wasp/out/Dockerfile` to see the new Dockerfile and what changes you need to make. You'll probably need to copy some of the changes from the new Dockerfile to your custom one to make your app work with Wasp 0.12.X.

### Migrating to the New Auth[​](#migrating-to-the-new-auth "Direct link to Migrating to the New Auth")

As shown in [the previous section](#new-auth), Wasp significantly changed how authentication works in version 0.12.0. This section leads you through migrating your app from Wasp 0.11.X to Wasp 0.12.X.

Migrating your existing app to the new auth system is a two-step process:

1.  Migrate to the new auth system
2.  Clean up the old auth system

Migrating a deployed app

While going through these steps, we will focus first on doing the changes locally (including your local development database).

Once we confirm everything works well locally, we will apply the same changes to the deployed app (including your production database).

**We'll put extra info for migrating a deployed app in a box like this one.**

#### 1\. Migrate to the New Auth System[​](#1-migrate-to-the-new-auth-system "Direct link to 1. Migrate to the New Auth System")

You can follow these steps to migrate to the new auth system (assuming you already migrated the project structure to 0.12, as described [above](#migrating-your-project-to-the-new-structure)):

1.  **Migrate `getUserFields` and/or `additionalSignupFields` in the `main.wasp` file to the new `userSignupFields` field.**
    
    If you are not using them, you can skip this step.
    
    In Wasp 0.11.X, you could define a `getUserFieldsFn` to specify extra fields that would get saved to the `User` when using Google or GitHub to sign up.
    
    You could also define `additionalSignupFields` to specify extra fields for the Email or Username & Password signup.
    
    In 0.12.X, we unified these two concepts into the `userSignupFields` field.
    
    Migration for [Email](/docs/auth/email) and [Username & Password](/docs/auth/username-and-pass)
    
    First, move the value of `auth.signup.additionalFields` to `auth.methods.{method}.userSignupFields` in the `main.wasp` file.
    
    `{method}` depends on the auth method you are using. For example, if you are using the email auth method, you should move the `auth.signup.additionalFields` to `auth.methods.email.userSignupFields`.
    
    To finish, update the JS/TS implementation to use the `defineUserSignupFields` from `wasp/server/auth` instead of `defineAdditionalSignupFields` from `@wasp/auth/index.js`.
    
    -   Before
    -   After
    
    main.wasp
    
    ```
        app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      email: {},    },    onAuthFailedRedirectTo: "/login",    signup: {      additionalFields: import { fields } from "@server/auth/signup.js",    },  },}
    ```
    
    src/server/auth/signup.ts
    
    ```
        import { defineAdditionalSignupFields } from '@wasp/auth/index.js'export const fields = defineAdditionalSignupFields({  address: async (data) => {    const address = data.address    if (typeof address !== 'string') {      throw new Error('Address is required')    }    if (address.length < 5) {      throw new Error('Address must be at least 5 characters long')    }    return address  },})
    ```
    
    main.wasp
    
    ```
        app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      email: {        userSignupFields: import { fields } from "@src/server/auth/signup.js",      },    },    onAuthFailedRedirectTo: "/login",  },}
    ```
    
    src/server/auth/signup.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const fields = defineUserSignupFields({  address: async (data) => {    const address = data.address;    if (typeof address !== 'string') {      throw new Error('Address is required');    }    if (address.length < 5) {      throw new Error('Address must be at least 5 characters long');    }    return address;  },})
    ```
    
    Read more about the `userSignupFields` function [here](/docs/auth/overview#1-defining-extra-fields).
    
    Migration for [Github](/docs/auth/social-auth/github) and [Google](/docs/auth/social-auth/google)
    
    First, move the value of `auth.methods.{method}.getUserFieldsFn` to `auth.methods.{method}.userSignupFields` in the `main.wasp` file.
    
    `{method}` depends on the auth method you are using. For example, if you are using Google auth, you should move the `auth.methods.google.getUserFieldsFn` to `auth.methods.google.userSignupFields`.
    
    To finish, update the JS/TS implementation to use the `defineUserSignupFields` from `wasp/server/auth` and modify the code to return the fields in the format that `defineUserSignupFields` expects.
    
    -   Before
    -   After
    
    main.wasp
    
    ```
        app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      google: {        getUserFieldsFn: import { getUserFields } from "@server/auth/google.js"      },    },    onAuthFailedRedirectTo: "/login",  },}
    ```
    
    src/server/auth/google.ts
    
    ```
        import type { GetUserFieldsFn } from '@wasp/types'export const getUserFields: GetUserFieldsFn = async (_context, args) => {  const displayName = args.profile.displayName  return { displayName }}
    ```
    
    main.wasp
    
    ```
        app crudTesting {  // ...  auth: {    userEntity: User,    methods: {      google: {        userSignupFields: import { fields } from "@src/server/auth/google.js",      },    },    onAuthFailedRedirectTo: "/login",  },}
    ```
    
    src/server/auth/signup.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const fields = defineUserSignupFields({  displayName: async (data) => {    const profile: any = data.profile;    if (!profile?.displayName) { throw new Error('Display name is not available'); }    return profile.displayName;  },})
    ```
    
    If you want to properly type the `profile` object, we recommend you use a validation library like Zod to define the shape of the `profile` object.
    
    Read more about this and the `defineUserSignupFields` function in the [Auth Overview - Defining Extra Fields](/docs/auth/overview#1-defining-extra-fields) section.
    
2.  **Remove the `auth.methods.email.allowUnverifiedLogin` field** from your `main.wasp` file.
    
    In Wasp 0.12.X we removed the `auth.methods.email.allowUnverifiedLogin` field to make our Email auth implementation easier to reason about. If you were using it, you should remove it from your `main.wasp` file.
    
3.  Ensure your **local development database is running**.
    
4.  **Do the schema migration** (create the new auth tables in the database) by running:
    
    ```
        wasp db migrate-dev
    ```
    
    You should see the new `Auth`, `AuthIdentity` and `Session` tables in your database. You can use the `wasp db studio` command to open the database in a GUI and verify the tables are there. At the moment, they will be empty.
    
5.  **Do the data migration** (move existing users from the old auth system to the new one by filling the new auth tables in the database with their data):
    
    1.  **Implement your data migration function(s)** in e.g. `src/migrateToNewAuth.ts`.
        
        Below we prepared [examples of migration functions](#example-data-migration-functions) for each of the auth methods, for you to use as a starting point. They should be fine to use as-is, meaning you can just copy them and they are likely to work out of the box for typical use cases, but you can also modify them for your needs.
        
        We recommend you create one function per each auth method that you use in your app.
        
    2.  **Define custom API endpoints for each migration function** you implemented.
        
        With each data migration function below, we provided a relevant `api` declaration that you should add to your `main.wasp` file.
        
    3.  **Run the data migration function(s)** on the local development database by calling the API endpoints you defined in the previous step.
        
        You can call the endpoint by visiting the URL in your browser, or by using a tool like `curl` or Postman.
        
        For example, if you defined the API endpoint at `/migrate-username-and-password`, you can call it by visiting `http://localhost:3001/migrate-username-and-password` in your browser.
        
        This should be it, you can now run `wasp db studio` again and verify that there is now relevant data in the new auth tables (`Auth` and `AuthIdentity`; `Session` should still be empty for now).
        
6.  **Verify that the basic auth functionality works** by running `wasp start` and successfully signing up / logging in with each of the auth methods.
    
7.  **Update your JS/TS code** to work correctly with the new auth.
    
    You might want to use the new auth helper functions to get the `email` or `username` from a user object. For example, `user.username` might not work anymore for you, since the `username` obtained by the Username & Password auth method isn't stored on the `User` entity anymore (unless you are explicitly storing something into `user.username`, e.g. via `userSignupFields` for a social auth method like Github). Same goes for `email` from Email auth method.
    
    Instead, you can now use `getUsername(user)` to get the username obtained from Username & Password auth method, or `getEmail(user)` to get the email obtained from Email auth method.
    
    Read more about the helpers in the [Accessing User Data](/docs/auth/entities#accessing-the-auth-fields) section.
    
8.  Finally, **check that your app now fully works as it worked before**. If all the above steps were done correctly, everything should be working now.
    
    Migrating a deployed app
    
    After successfully performing migration locally so far, and verifying that your app works as expected, it is time to also migrate our deployed app.
    
    Before migrating your production (deployed) app, we advise you to back up your production database in case something goes wrong. Also, besides testing it in development, it's good to test the migration in a staging environment if you have one.
    
    We will perform the production migration in 2 steps:
    
    -   Deploying the new code to production (client and server).
    -   Migrating the production database data.
    
    ---
    
    Between these two steps, so after successfully deploying the new code to production and before migrating the production database data, your app will not be working completely: new users will be able to sign up, but existing users won't be able to log in, and already logged in users will be logged out. Once you do the second step, migrating the production database data, it will all be back to normal. You will likely want to keep the time between the two steps as short as you can.
    
    ---
    
    -   **First step: deploy the new code** (client and server), either via `wasp deploy` (i.e. `wasp deploy fly deploy`) or manually.
        
        Check our [Deployment docs](/docs/advanced/deployment/overview) for more details.
        
    -   **Second step: run the data migration functions** on the production database.
        
        You can do this by calling the API endpoints you defined in the previous step, just like you did locally. You can call the endpoint by visiting the URL in your browser, or by using a tool like `curl` or Postman.
        
        For example, if you defined the API endpoint at `/migrate-username-and-password`, you can call it by visiting `https://your-server-url.com/migrate-username-and-password` in your browser.
        
    
    Your deployed app should be working normally now, with the new auth system.
    

#### 2\. Cleanup the Old Auth System[​](#2-cleanup-the-old-auth-system "Direct link to 2. Cleanup the Old Auth System")

Your app should be working correctly and using new auth, but to finish the migration, we need to clean up the old auth system:

1.  In `main.wasp` file, **delete auth-related fields from the `User` entity**, since with 0.12 they got moved to the internal Wasp entity `AuthIdentity`.
    
    -   This means any fields that were required by Wasp for authentication, like `email`, `password`, `isEmailVerified`, `emailVerificationSentAt`, `passwordResetSentAt`, `username`, etc.
    -   There are situations in which you might want to keep some of them, e.g. `email` and/or `username`, if they are still relevant for you due to your custom logic (e.g. you are populating them with `userSignupFields` upon social signup in order to have this info easily available on the `User` entity). Note that they won't be used by Wasp Auth anymore, they are here just for your business logic.
2.  In `main.wasp` file, **remove the `externalAuthEntity` field from the `app.auth`** and also **remove the whole `SocialLogin` entity** if you used Google or GitHub auth.
    
3.  **Delete the data migration function(s)** you implemented earlier (e.g. in `src/migrateToNewAuth.ts`) and also the corresponding API endpoints from the `main.wasp` file.
    
4.  **Run `wasp db migrate-dev`** again to apply these changes and remove the redundant fields from the database.
    

Migrating a deployed app

After doing the steps above successfully locally and making sure everything is working, it is time to push these changes to the deployed app again.

*Deploy the app again*, either via `wasp deploy` or manually. Check our [Deployment docs](/docs/advanced/deployment/overview) for more details.

The database migrations will automatically run on successful deployment of the server and delete the now redundant auth-related `User` columns from the database.

Your app is now fully migrated to the new auth system.

### Next Steps[​](#next-steps "Direct link to Next Steps")

If you made it this far, you've completed all the necessary steps to get your Wasp app working with Wasp 0.12.x. Nice work!

Finally, since Wasp no longer requires you to separate your client source files (previously in `src/client`) from server source files (previously in `src/server`), you are now free to reorganize your project however you think is best, as long as you keep all the source files in the `src/` directory.

This section is optional, but if you didn't like the server/client separation, now's the perfect time to change it.

For example, if your `src` dir looked like this:

```
    src│├── client│   ├── Dashboard.tsx│   ├── Login.tsx│   ├── MainPage.tsx│   ├── Register.tsx│   ├── Task.css│   ├── TaskLisk.tsx│   ├── Task.tsx│   └── User.tsx├── server│   ├── taskActions.ts│   ├── taskQueries.ts│   ├── userActions.ts│   └── userQueries.ts└── shared    └── utils.ts
```

you can now change it to a feature-based structure (which we recommend for any project that is not very small):

```
    src│├── task│   ├── actions.ts    -- former taskActions.ts│   ├── queries.ts    -- former taskQueries.ts│   ├── Task.css│   ├── TaskLisk.tsx│   └── Task.tsx├── user│   ├── actions.ts    -- former userActions.ts│   ├── Dashboard.tsx│   ├── Login.tsx│   ├── queries.ts    -- former userQueries.ts│   ├── Register.tsx│   └── User.tsx├── MainPage.tsx└── utils.ts
```

## Appendix[​](#appendix "Direct link to Appendix")

### Example Data Migration Functions[​](#example-data-migration-functions "Direct link to Example Data Migration Functions")

The migration functions provided below are written with the typical use cases in mind and you can use them as-is. If your setup requires additional logic, you can use them as a good starting point and modify them to your needs.

Note that all of the functions below are written to be idempotent, meaning that running a function multiple times can't hurt. This allows executing a function again in case only a part of the previous execution succeeded and also means that accidentally running it one time too much won't have any negative effects. **We recommend you keep your data migration functions idempotent**.

#### Username & Password[​](#username--password "Direct link to Username & Password")

To successfully migrate the users using the Username & Password auth method, you will need to do two things:

1.  Migrate the user data
    
    Username & Password data migration function
    
    main.wasp
    
    ```
        api migrateUsernameAndPassword {  httpRoute: (GET, "/migrate-username-and-password"),  fn: import { migrateUsernameAndPasswordHandler } from "@src/migrateToNewAuth",  entities: []}
    ```
    
    src/migrateToNewAuth.ts
    
    ```
        import { prisma } from "wasp/server";import { type ProviderName, type UsernameProviderData } from "wasp/server/auth";import { MigrateUsernameAndPassword } from "wasp/server/api";export const migrateUsernameAndPasswordHandler: MigrateUsernameAndPassword =  async (_req, res) => {    const result = await migrateUsernameAuth();    res.status(200).json({ message: "Migrated users to the new auth", result });  };async function migrateUsernameAuth(): Promise<{  numUsersAlreadyMigrated: number;  numUsersNotUsingThisAuthMethod: number;  numUsersMigratedSuccessfully: number;}> {  const users = await prisma.user.findMany({    include: {      auth: true,    },  });  const result = {    numUsersAlreadyMigrated: 0,    numUsersNotUsingThisAuthMethod: 0,    numUsersMigratedSuccessfully: 0,  };  for (const user of users) {    if (user.auth) {      result.numUsersAlreadyMigrated++;      console.log("Skipping user (already migrated) with id:", user.id);      continue;    }    if (!user.username || !user.password) {      result.numUsersNotUsingThisAuthMethod++;      console.log("Skipping user (not using username auth) with id:", user.id);      continue;    }    const providerData: UsernameProviderData = {      hashedPassword: user.password,    };    const providerName: ProviderName = "username";    await prisma.auth.create({      data: {        identities: {          create: {            providerName,            providerUserId: user.username.toLowerCase(),            providerData: JSON.stringify(providerData),          },        },        user: {          connect: {            id: user.id,          },        },      },    });    result.numUsersMigratedSuccessfully++;  }  return result;}
    ```
    
2.  Provide a way for users to migrate their password
    
    There is a **breaking change between the old and the new auth in the way the password is hashed**. This means that users will need to migrate their password after the migration, as the old password will no longer work.
    
    Since the only way users using username and password as a login method can verify their identity is by providing both their username and password (there is no email or any other info, unless you asked for it and stored it explicitly), we need to provide them a way to **exchange their old password for a new password**. One way to handle this is to inform them about the need to migrate their password (on the login page) and provide a custom page to migrate the password.
    

Steps to create a custom page for migrating the password

1.  You will need to install the `secure-password` and `sodium-native` packages to use the old hashing algorithm:
    
    ```
        npm install secure-password@4.0.0 sodium-native@3.3.0 --save-exact
    ```
    
    Make sure to save the exact versions of the packages.
    
2.  Then you'll need to create a new page in your app where users can migrate their password. You can use the following code as a starting point:
    

-   JavaScript
-   TypeScript

main.wasp

```
    route MigratePasswordRoute { path: "/migrate-password", to: MigratePassword }page MigratePassword {  component: import { MigratePasswordPage } from "@src/pages/MigratePassword"}
```

src/pages/MigratePassword.jsx

```
    import {  FormItemGroup,  FormLabel,  FormInput,  FormError,} from "wasp/client/auth";import { useForm } from "react-hook-form";import { migratePassword } from "wasp/client/operations";import { useState } from "react";export function MigratePasswordPage() {  const [successMessage, setSuccessMessage] = useState(null);  const [errorMessage, setErrorMessage] = useState(null);  const form = useForm();  const onSubmit = form.handleSubmit(async (data) => {    try {      const result = await migratePassword(data);      setSuccessMessage(result.message);    } catch (e) {      console.error(e);      if (e instanceof Error) {        setErrorMessage(e.message);      }    }  });  return (    <div style={{      maxWidth: "400px",      margin: "auto",    }}>      <h1>Migrate your password</h1>      <p>        If you have an account on the old version of the website, you can        migrate your password to the new version.      </p>      {successMessage && <div>{successMessage}</div>}      {errorMessage && <FormError>{errorMessage}</FormError>}      <form onSubmit={onSubmit}>        <FormItemGroup>          <FormLabel>Username</FormLabel>          <FormInput            {...form.register("username", {              required: "Username is required",            })}          />          <FormError>{form.formState.errors.username?.message}</FormError>        </FormItemGroup>        <FormItemGroup>          <FormLabel>Password</FormLabel>          <FormInput            {...form.register("password", {              required: "Password is required",            })}            type="password"          />          <FormError>{form.formState.errors.password?.message}</FormError>        </FormItemGroup>        <button type="submit">Migrate password</button>      </form>    </div>  );}
```

main.wasp

```
    route MigratePasswordRoute { path: "/migrate-password", to: MigratePassword }page MigratePassword {  component: import { MigratePasswordPage } from "@src/pages/MigratePassword"}
```

src/pages/MigratePassword.tsx

```
    import {  FormItemGroup,  FormLabel,  FormInput,  FormError,} from "wasp/client/auth";import { useForm } from "react-hook-form";import { migratePassword } from "wasp/client/operations";import { useState } from "react";export function MigratePasswordPage() {  const [successMessage, setSuccessMessage] = useState<string | null>(null);  const [errorMessage, setErrorMessage] = useState<string | null>(null);  const form = useForm<{    username: string;    password: string;  }>();  const onSubmit = form.handleSubmit(async (data) => {    try {      const result = await migratePassword(data);      setSuccessMessage(result.message);    } catch (e: unknown) {      console.error(e);      if (e instanceof Error) {        setErrorMessage(e.message);      }    }  });  return (    <div style={{      maxWidth: "400px",      margin: "auto",    }}>      <h1>Migrate your password</h1>      <p>        If you have an account on the old version of the website, you can        migrate your password to the new version.      </p>      {successMessage && <div>{successMessage}</div>}      {errorMessage && <FormError>{errorMessage}</FormError>}      <form onSubmit={onSubmit}>        <FormItemGroup>          <FormLabel>Username</FormLabel>          <FormInput            {...form.register("username", {              required: "Username is required",            })}          />          <FormError>{form.formState.errors.username?.message}</FormError>        </FormItemGroup>        <FormItemGroup>          <FormLabel>Password</FormLabel>          <FormInput            {...form.register("password", {              required: "Password is required",            })}            type="password"          />          <FormError>{form.formState.errors.password?.message}</FormError>        </FormItemGroup>        <button type="submit">Migrate password</button>      </form>    </div>  );}
```

3.  Finally, you will need to create a new operation in your app to handle the password migration. You can use the following code as a starting point:

-   JavaScript
-   TypeScript

main.wasp

```
    action migratePassword {  fn: import { migratePassword } from "@src/auth",  entities: []}
```

src/auth.js

```
    import SecurePassword from "secure-password";import { HttpError } from "wasp/server";import {  createProviderId,  deserializeAndSanitizeProviderData,  findAuthIdentity,  updateAuthIdentityProviderData,} from "wasp/server/auth";export const migratePassword = async ({ password, username }, _context) => {  const providerId = createProviderId("username", username);  const authIdentity = await findAuthIdentity(providerId);  if (!authIdentity) {    throw new HttpError(400, "Something went wrong");  }  const providerData = deserializeAndSanitizeProviderData(    authIdentity.providerData  );  try {    const SP = new SecurePassword();    // This will verify the password using the old algorithm    const result = await SP.verify(      Buffer.from(password),      Buffer.from(providerData.hashedPassword, "base64")    );    if (result !== SecurePassword.VALID) {      throw new HttpError(400, "Something went wrong");    }    // This will hash the password using the new algorithm and update the    // provider data in the database.    await updateAuthIdentityProviderData(providerId, providerData, {      hashedPassword: password,    });  } catch (e) {    throw new HttpError(400, "Something went wrong");  }  return {    message: "Password migrated successfully.",  };};
```

main.wasp

```
    action migratePassword {  fn: import { migratePassword } from "@src/auth",  entities: []}
```

src/auth.ts

```
    import SecurePassword from "secure-password";import { HttpError } from "wasp/server";import {  createProviderId,  deserializeAndSanitizeProviderData,  findAuthIdentity,  updateAuthIdentityProviderData,} from "wasp/server/auth";import { MigratePassword } from "wasp/server/operations";type MigratePasswordInput = {  username: string;  password: string;};type MigratePasswordOutput = {  message: string;};export const migratePassword: MigratePassword<  MigratePasswordInput,  MigratePasswordOutput> = async ({ password, username }, _context) => {  const providerId = createProviderId("username", username);  const authIdentity = await findAuthIdentity(providerId);  if (!authIdentity) {    throw new HttpError(400, "Something went wrong");  }  const providerData = deserializeAndSanitizeProviderData<"username">(    authIdentity.providerData  );  try {    const SP = new SecurePassword();    // This will verify the password using the old algorithm    const result = await SP.verify(      Buffer.from(password),      Buffer.from(providerData.hashedPassword, "base64")    );    if (result !== SecurePassword.VALID) {      throw new HttpError(400, "Something went wrong");    }    // This will hash the password using the new algorithm and update the    // provider data in the database.    await updateAuthIdentityProviderData<"username">(providerId, providerData, {      hashedPassword: password,    });  } catch (e) {    throw new HttpError(400, "Something went wrong");  }  return {    message: "Password migrated successfully.",  };};
```

#### Email[​](#email "Direct link to Email")

To successfully migrate the users using the Email auth method, you will need to do two things:

1.  Migrate the user data
    
    Email data migration function
    
    main.wasp
    
    ```
        api migrateEmail {  httpRoute: (GET, "/migrate-email"),  fn: import { migrateEmailHandler } from "@src/migrateToNewAuth",  entities: []}
    ```
    
    src/migrateToNewAuth.ts
    
    ```
        import { prisma } from "wasp/server";import { type ProviderName, type EmailProviderData } from "wasp/server/auth";import { MigrateEmail } from "wasp/server/api";export const migrateEmailHandler: MigrateEmail =  async (_req, res) => {    const result = await migrateEmailAuth();    res.status(200).json({ message: "Migrated users to the new auth", result });  };async function migrateEmailAuth(): Promise<{  numUsersAlreadyMigrated: number;  numUsersNotUsingThisAuthMethod: number;  numUsersMigratedSuccessfully: number;}> {  const users = await prisma.user.findMany({    include: {      auth: true,    },  });  const result = {    numUsersAlreadyMigrated: 0,    numUsersNotUsingThisAuthMethod: 0,    numUsersMigratedSuccessfully: 0,  };  for (const user of users) {    if (user.auth) {      result.numUsersAlreadyMigrated++;      console.log("Skipping user (already migrated) with id:", user.id);      continue;    }    if (!user.email || !user.password) {      result.numUsersNotUsingThisAuthMethod++;      console.log("Skipping user (not using email auth) with id:", user.id);      continue;    }    const providerData: EmailProviderData = {      isEmailVerified: user.isEmailVerified,      emailVerificationSentAt:        user.emailVerificationSentAt?.toISOString() ?? null,      passwordResetSentAt: user.passwordResetSentAt?.toISOString() ?? null,      hashedPassword: user.password,    };    const providerName: ProviderName = "email";    await prisma.auth.create({      data: {        identities: {          create: {            providerName,            providerUserId: user.email,            providerData: JSON.stringify(providerData),          },        },        user: {          connect: {            id: user.id,          },        },      },    });    result.numUsersMigratedSuccessfully++;  }  return result;}
    ```
    
2.  Ask the users to reset their password
    
    There is a **breaking change between the old and the new auth in the way the password is hashed**. This means that users will need to reset their password after the migration, as the old password will no longer work.
    
    It would be best to notify your users about this change and put a notice on your login page to **request a password reset**.
    

#### Google & GitHub[​](#google--github "Direct link to Google & GitHub")

Google & GitHub data migration functions

main.wasp

```
    api migrateGoogle {  httpRoute: (GET, "/migrate-google"),  fn: import { migrateGoogleHandler } from "@src/migrateToNewAuth",  entities: []}api migrateGithub {  httpRoute: (GET, "/migrate-github"),  fn: import { migrateGithubHandler } from "@src/migrateToNewAuth",  entities: []}
```

src/migrateToNewAuth.ts

```
    import { prisma } from "wasp/server";import { MigrateGoogle, MigrateGithub } from "wasp/server/api";export const migrateGoogleHandler: MigrateGoogle =  async (_req, res) => {    const result = await createSocialLoginMigration("google");    res.status(200).json({ message: "Migrated users to the new auth", result });  };export const migrateGithubHandler: MigrateGithub =  async (_req, res) => {    const result = await createSocialLoginMigration("github");    res.status(200).json({ message: "Migrated users to the new auth", result });  };async function createSocialLoginMigration(  providerName: "google" | "github"): Promise<{  numUsersAlreadyMigrated: number;  numUsersNotUsingThisAuthMethod: number;  numUsersMigratedSuccessfully: number;}> {  const users = await prisma.user.findMany({    include: {      auth: true,      externalAuthAssociations: true,    },  });  const result = {    numUsersAlreadyMigrated: 0,    numUsersNotUsingThisAuthMethod: 0,    numUsersMigratedSuccessfully: 0,  };  for (const user of users) {    if (user.auth) {      result.numUsersAlreadyMigrated++;      console.log("Skipping user (already migrated) with id:", user.id);      continue;    }    const provider = user.externalAuthAssociations.find(      (provider) => provider.provider === providerName    );    if (!provider) {      result.numUsersNotUsingThisAuthMethod++;      console.log(`Skipping user (not using ${providerName} auth) with id:`, user.id);      continue;    }    await prisma.auth.create({      data: {        identities: {          create: {            providerName,            providerUserId: provider.providerId,            providerData: JSON.stringify({}),          },        },        user: {          connect: {            id: user.id,          },        },      },    });    result.numUsersMigratedSuccessfully++;  }  return result;}
```

---

# Migration from 0.12.X to 0.13.X

Are you on 0.11.X or earlier?

This guide only covers the migration from **0.12.X to 0.13.X**. If you are migrating from 0.11.X or earlier, please read the [migration guide from 0.11.X to 0.12.X](/docs/migrate-from-0-11-to-0-12) first.

Make sure to read the [migration guide from 0.13.X to 0.14.X](/docs/migrate-from-0-13-to-0-14) after you finish this one.

## What's new in 0.13.0?[​](#whats-new-in-0130 "Direct link to What's new in 0.13.0?")

### OAuth providers got an overhaul[​](#oauth-providers-got-an-overhaul "Direct link to OAuth providers got an overhaul")

Wasp 0.13.0 switches away from using Passport for our OAuth providers in favor of [Arctic](https://arctic.js.org/) from the [Lucia](https://lucia-auth.com/) ecosystem. This change simplifies the codebase and makes it easier to add new OAuth providers in the future.

### We added Keycloak as an OAuth provider[​](#we-added-keycloak-as-an-oauth-provider "Direct link to We added Keycloak as an OAuth provider")

Wasp now supports using [Keycloak](https://www.keycloak.org/) as an OAuth provider.

## How to migrate?[​](#how-to-migrate "Direct link to How to migrate?")

### Migrate your OAuth setup[​](#migrate-your-oauth-setup "Direct link to Migrate your OAuth setup")

We had to make some breaking changes to upgrade the OAuth setup to the new Arctic lib.

Follow the steps below to migrate:

1.  **Define the `WASP_SERVER_URL` server env variable**
    
    In 0.13.0 Wasp introduces a new server env variable `WASP_SERVER_URL` that you need to define. This is the URL of your Wasp server and it's used to generate the redirect URL for the OAuth providers.
    
    Server env variables
    
    ```
        WASP_SERVER_URL=https://your-wasp-server-url.com
    ```
    
    In development, Wasp sets the `WASP_SERVER_URL` to `http://localhost:3001` by default.
    
    Migrating a deployed app
    
    If you are migrating a deployed app, you will need to define the `WASP_SERVER_URL` server env variable in your deployment environment.
    
    Read more about setting env variables in production [here](/docs/project/env-vars#defining-env-vars-in-production).
    
2.  **Update the redirect URLs** for the OAuth providers
    
    The redirect URL for the OAuth providers has changed. You will need to update the redirect URL for the OAuth providers in the provider's dashboard.
    
    -   Before
    -   After
    
    ```
        {clientUrl}/auth/login/{provider}
    ```
    
    ```
        {serverUrl}/auth/{provider}/callback
    ```
    
    Check the new redirect URLs for [Google](/docs/auth/social-auth/google#3-creating-a-google-oauth-app) and [GitHub](/docs/auth/social-auth/github#3-creating-a-github-oauth-app) in Wasp's docs.
    
3.  **Update the `configFn`** for the OAuth providers
    
    If you didn't use the `configFn` option, you can skip this step.
    
    If you used the `configFn` to configure the `scope` for the OAuth providers, you will need to rename the `scope` property to `scopes`.
    
    Also, the object returned from `configFn` no longer needs to include the Client ID and the Client Secret. You can remove them from the object that `configFn` returns.
    
    -   Before
    -   After
    
    google.ts
    
    ```
        export function getConfig() {    return {        clientID: process.env.GOOGLE_CLIENT_ID,        clientSecret: process.env.GOOGLE_CLIENT_SECRET,        scope: ['profile', 'email'],    }}
    ```
    
    google.ts
    
    ```
        export function getConfig() {    return {        scopes: ['profile', 'email'],    }}
    ```
    
4.  **Update the `userSignupFields` fields** to use the new `profile` format
    
    If you didn't use the `userSignupFields` option, you can skip this step.
    
    The data format for the `profile` that you receive from the OAuth providers has changed. You will need to update your code to reflect this change.
    
    -   Before
    -   After
    
    google.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({    displayName: (data: any) => data.profile.displayName,})
    ```
    
    google.ts
    
    ```
        import { defineUserSignupFields } from 'wasp/server/auth'export const userSignupFields = defineUserSignupFields({    displayName: (data: any) => data.profile.name,})
    ```
    
    Wasp now directly forwards what it receives from the OAuth providers. You can check the data format for [Google](/docs/auth/social-auth/google#data-received-from-google) and [GitHub](/docs/auth/social-auth/github#data-received-from-github) in Wasp's docs.
    

That's it!

You should now be able to run your app with the new Wasp 0.13.0.

---

# Migration from 0.13.X to 0.14.X

Are you on 0.11.X or earlier?

This guide only covers the migration from **0.13.X to 0.14.X**. If you are migrating from 0.11.X or earlier, please read the [migration guide from 0.11.X to 0.12.X](/docs/migrate-from-0-11-to-0-12) first.

## What's new in 0.14.0?[​](#whats-new-in-0140 "Direct link to What's new in 0.14.0?")

### Using Prisma Schema file directly[​](#using-prisma-schema-file-directly "Direct link to Using Prisma Schema file directly")

Before 0.14.0, users defined their entities in the `.wasp` file, and Wasp generated the `schema.prisma` file based on that. This approach had some limitations, and users couldn't use some advanced Prisma features.

Wasp now exposes the `schema.prisma` file directly to the user. You now define your entities in the `schema.prisma` file and Wasp uses that to generate the database schema and Prisma client. You can use all the Prisma features directly in the `schema.prisma` file. Simply put, the `schema.prisma` file is now the source of truth for your database schema.

-   Before
-   After

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "MyApp",  db: {    system: PostgreSQL  },}entity User {=psl  id       Int @id @default(autoincrement())  tasks    Task[]psl=}entity Task {=psl  id          Int @id @default(autoincrement())  description String  isDone      Boolean  userId      Int  user        User @relation(fields: [userId], references: [id])psl=}
```

main.wasp

```
    app myApp {  wasp: {    version: "^0.13.0"  },  title: "MyApp",}
```

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}generator client {  provider = "prisma-client-js"}model User {  id       Int @id @default(autoincrement())  tasks    Task[]}model Task {  id          Int @id @default(autoincrement())  description String  isDone      Boolean  userId      Int  user        User @relation(fields: [userId], references: [id])}
```

### Better auth user API[​](#better-auth-user-api "Direct link to Better auth user API")

Wasp introduced a much simpler API for accessing user auth fields like `username`, `email` or `isEmailVerified` on the `user` object. You don't need to use helper functions every time you want to access the user's `username` or do extra steps to get proper typing.

## How to migrate?[​](#how-to-migrate "Direct link to How to migrate?")

To migrate your app to Wasp 0.14.x, you must:

1.  Bump the version in `main.wasp` and update your `tsconfig.json`.
2.  Migrate your entities into the new `schema.prisma` file.
3.  Update code that accesses user fields.

### Bump the version and update `tsconfig.json`[​](#bump-the-version-and-update-tsconfigjson "Direct link to bump-the-version-and-update-tsconfigjson")

Let's start with something simple. Update the version field in your Wasp file to `^0.14.0`:

main.wasp

```
    app MyApp {  wasp: {    version: "^0.14.0"  },}
```

To ensure your project works correctly with Wasp 0.14.0, you must also update your `tsconfig.json` file.

If you haven't changed anything in your project's `tsconfig.json` file (this is the case for most users), just replace its contents with the new version shown below.

If you have made changes to your `tsconfig.json` file, we recommend taking the new version of the file and reapplying them.

Here's the new version of the `tsconfig.json` file:

tsconfig.json

```
    // =============================== IMPORTANT =================================//// This file is only used for Wasp IDE support. You can change it to configure// your IDE checks, but none of these options will affect the TypeScript// compiler. Proper TS compiler configuration in Wasp is coming soon :){  "compilerOptions": {    "module": "esnext",    "target": "esnext",    // We're bundling all code in the end so this is the most appropriate option,    // it's also important for autocomplete to work properly.    "moduleResolution": "bundler",    // JSX support    "jsx": "preserve",    "strict": true,    // Allow default imports.    "esModuleInterop": true,    "lib": ["dom", "dom.iterable", "esnext"],    "allowJs": true,    "typeRoots": [      // This is needed to properly support Vitest testing with jest-dom matchers.      // Types for jest-dom are not recognized automatically and Typescript complains      // about missing types e.g. when using `toBeInTheDocument` and other matchers.      "node_modules/@testing-library",      // Specifying type roots overrides the default behavior of looking at the      // node_modules/@types folder so we had to list it explicitly.      // Source 1: https://www.typescriptlang.org/tsconfig#typeRoots      // Source 2: https://github.com/testing-library/jest-dom/issues/546#issuecomment-1889884843      "node_modules/@types"    ],    // Since this TS config is used only for IDE support and not for    // compilation, the following directory doesn't exist. We need to specify    // it to prevent this error:    // https://stackoverflow.com/questions/42609768/typescript-error-cannot-write-file-because-it-would-overwrite-input-file    "outDir": ".wasp/phantom"  }}
```

### Migrate to the new `schema.prisma` file[​](#migrate-to-the-new-schemaprisma-file "Direct link to migrate-to-the-new-schemaprisma-file")

To use the new `schema.prisma` file, you need to move your entities from the `.wasp` file to the `schema.prisma` file.

1. **Create a new `schema.prisma` file**

Create a new file named `schema.prisma` in the root of your project:

```
    .├── main.wasp...├── schema.prisma├── src├── tsconfig.json└── vite.config.ts
```

2. **Add the `datasource` block** to the `schema.prisma` file

This block specifies the database type and connection URL:

-   Sqlite
-   PostgreSQL

schema.prisma

```
    datasource db {  provider = "sqlite"  url      = env("DATABASE_URL")}
```

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}
```

-   The `provider` should be either `"postgresql"` or `"sqlite"`.
    
-   The `url` must be set to `env("DATABASE_URL")` so that Wasp can inject the database URL from the environment variables.
    

3. **Add the `generator` block** to the `schema.prisma` file

This block specifies the Prisma Client generator Wasp uses:

-   Sqlite
-   PostgreSQL

schema.prisma

```
    datasource db {  provider = "sqlite"  url      = env("DATABASE_URL")}generator client {  provider = "prisma-client-js"}
```

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}generator client {  provider = "prisma-client-js"}
```

-   The `provider` should be set to `"prisma-client-js"`.

4. **Move your entities** to the `schema.prisma` file

Move the entities from the `.wasp` file to the `schema.prisma` file:

-   Sqlite
-   PostgreSQL

schema.prisma

```
    datasource db {  provider = "sqlite"  url      = env("DATABASE_URL")}generator client {  provider = "prisma-client-js"}// There are some example entities, you should move your entities heremodel User {  id       Int @id @default(autoincrement())  tasks    Task[]}model Task {  id          Int @id @default(autoincrement())  description String  isDone      Boolean  userId      Int  user        User @relation(fields: [userId], references: [id])}
```

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")}generator client {  provider = "prisma-client-js"}// There are some example entities, you should move your entities heremodel User {  id       Int @id @default(autoincrement())  tasks    Task[]}model Task {  id          Int @id @default(autoincrement())  description String  isDone      Boolean  userId      Int  user        User @relation(fields: [userId], references: [id])}
```

When moving the entities over, you'll need to change `entity` to `model` and remove the `=psl` and `psl=` tags.

If you had the following in the `.wasp` file:

main.wasp

```
    entity Task {=psl  // Stays the samepsl=}
```

... it would look like this in the `schema.prisma` file:

schema.prisma

```
    model Task {  // Stays the same}
```

5. **Remove `app.db.system`** field from the Wasp file

We now configure the DB system in the `schema.prisma` file, so there is no need for that field in the Wasp file.

main.wasp

```
    app MyApp {  // ...  db: {    system: PostgreSQL,  }}
```

6. **Migrate Prisma preview features config** to the `schema.prisma` file

If you didn't use any Prisma preview features, you can skip this step.

If you had the following in the `.wasp` file:

main.wasp

```
    app MyApp {  // ...  db: {    prisma: {      clientPreviewFeatures: ["postgresqlExtensions"]      dbExtensions: [        { name: "hstore", schema: "myHstoreSchema" },        { name: "pg_trgm" },        { name: "postgis", version: "2.1" },      ]    }  }}
```

... it will become this:

schema.prisma

```
    datasource db {  provider = "postgresql"  url      = env("DATABASE_URL")  extensions = [hstore(schema: "myHstoreSchema"), pg_trgm, postgis(version: "2.1")]}generator client {  provider = "prisma-client-js"  previewFeatures = ["postgresqlExtensions"]}
```

All that's left to do is migrate the database.

To avoid type errors, it's best to take care of database migrations after you've migrated the rest of the code. So, just keep reading, and we will remind you to migrate the database as [the last step of the migration guide](#migrate-the-database).

Read more about the [Prisma Schema File](/docs/data-model/prisma-file) and how Wasp uses it to generate the database schema and Prisma client.

### Migrate how you access user auth fields[​](#migrate-how-you-access-user-auth-fields "Direct link to Migrate how you access user auth fields")

We had to make a couple of breaking changes to reach the new simpler API.

Follow the steps below to migrate:

1.  **Replace the `getUsername` helper** with `user.identities.username.id`
    
    If you didn't use the `getUsername` helper in your code, you can skip this step.
    
    This helper changed and it no longer works with the `user` you receive as a prop on a page or through the `context`. You'll need to replace it with `user.identities.username.id`.
    
    -   Before
    -   After
    
    src/MainPage.tsx
    
    ```
        import { getUsername, AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const username = getUsername(user)  // ...}
    ```
    
    src/tasks.ts
    
    ```
        import { getUsername } from 'wasp/auth'export const createTask: CreateTask<...>  = async (args, context) => {    const username = getUsername(context.user)    // ...}
    ```
    
    src/MainPage.tsx
    
    ```
        import { AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const username = user.identities.username?.id  // ...}
    ```
    
    src/tasks.ts
    
    ```
        export const createTask: CreateTask<...>  = async (args, context) => {    const username = context.user.identities.username?.id    // ...}
    ```
    
2.  **Replace the `getEmail` helper** with `user.identities.email.id`
    
    If you didn't use the `getEmail` helper in your code, you can skip this step.
    
    This helper changed and it no longer works with the `user` you receive as a prop on a page or through the `context`. You'll need to replace it with `user.identities.email.id`.
    
    -   Before
    -   After
    
    src/MainPage.tsx
    
    ```
        import { getEmail, AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const email = getEmail(user)  // ...}
    ```
    
    src/tasks.ts
    
    ```
        import { getEmail } from 'wasp/auth'export const createTask: CreateTask<...>  = async (args, context) => {    const email = getEmail(context.user)    // ...}
    ```
    
    src/MainPage.tsx
    
    ```
        import { AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const email = user.identities.email?.id  // ...}
    ```
    
    src/tasks.ts
    
    ```
        export const createTask: CreateTask<...>  = async (args, context) => {    const email = context.user.identities.email?.id    // ...}
    ```
    
3.  **Replace accessing `providerData`** with `user.identities.<provider>.<value>`
    
    If you didn't use any data from the `providerData` object, you can skip this step.
    
    Replace `<provider>` with the provider name (for example `username`, `email`, `google`, `github`, etc.) and `<value>` with the field you want to access (for example `isEmailVerified`).
    
    -   Before
    -   After
    
    src/MainPage.tsx
    
    ```
        import { findUserIdentity, AuthUser } from 'wasp/auth'function getProviderData(user: AuthUser) {  const emailIdentity = findUserIdentity(user, 'email')  // We needed this before check for proper type support  return emailIdentity && 'isEmailVerified' in emailIdentity.providerData    ? emailIdentity.providerData    : null}const MainPage = ({ user }: { user: AuthUser }) => {  const providerData = getProviderData(user)  const isEmailVerified = providerData ? providerData.isEmailVerified : null  // ...}
    ```
    
    src/MainPage.tsx
    
    ```
        import { AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  // The email object is properly typed, so we can access `isEmailVerified` directly  const isEmailVerified = user.identities.email?.isEmailVerified  // ...}
    ```
    
4.  **Use `getFirstProviderUserId` directly** on the user object
    
    If you didn't use `getFirstProviderUserId` in your code, you can skip this step.
    
    You should replace `getFirstProviderUserId(user)` with `user.getFirstProviderUserId()`.
    
    -   Before
    -   After
    
    src/MainPage.tsx
    
    ```
        import { getFirstProviderUserId, AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const userId = getFirstProviderUserId(user)  // ...}
    ```
    
    src/tasks.ts
    
    ```
        import { getFirstProviderUserId } from 'wasp/auth'export const createTask: CreateTask<...>  = async (args, context) => {    const userId = getFirstProviderUserId(context.user)    // ...}
    ```
    
    src/MainPage.tsx
    
    ```
        import { AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const userId = user.getFirstProviderUserId()  // ...}
    ```
    
    src/tasks.ts
    
    ```
        export const createTask: CreateTask<...>  = async (args, context) => {    const userId = user.getFirstProviderUserId()    // ...}
    ```
    
5.  **Replace `findUserIdentity`** with checks on `user.identities.<provider>`
    
    If you didn't use `findUserIdentity` in your code, you can skip this step.
    
    Instead of using `findUserIdentity` to get the identity object, you can directly check if the identity exists on the `identities` object.
    
    -   Before
    -   After
    
    src/MainPage.tsx
    
    ```
        import { findUserIdentity, AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  const usernameIdentity = findUserIdentity(user, 'username')  if (usernameIdentity) {    // ...  }}
    ```
    
    src/tasks.ts
    
    ```
        import { findUserIdentity } from 'wasp/auth'export const createTask: CreateTask<...>  = async (args, context) => {    const usernameIdentity = findUserIdentity(context.user, 'username')    if (usernameIdentity) {        // ...    }}
    ```
    
    src/MainPage.tsx
    
    ```
        import { AuthUser } from 'wasp/auth'const MainPage = ({ user }: { user: AuthUser }) => {  if (user.identities.username) {    // ...  }}
    ```
    
    src/tasks.ts
    
    ```
        export const createTask: CreateTask<...>  = async (args, context) => {    if (context.user.identities.username) {        // ...    }}
    ```
    

### Migrate the database[​](#migrate-the-database "Direct link to Migrate the database")

Finally, you can **Run the Wasp CLI** to regenerate the new Prisma client:

```
    wasp db migrate-dev
```

This command generates the Prisma client based on the `schema.prisma` file.

Read more about the [Prisma Schema File](/docs/data-model/prisma-file) and how Wasp uses it to generate the database schema and Prisma client.

That's it!

You should now be able to run your app with the new Wasp 0.14.0. We recommend reading through the updated [Accessing User Data](/docs/auth/entities) section to get a better understanding of the new API.