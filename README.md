# 🚀 How I deploy to the permaweb

The permaweb is a cloud platform designed to publish and serve web artifacts from a decentralized permanent storage network. Using a permanent storage network, you pay once to post your application and content, and it will be available forever. The ability to pay once is great for developers and end-users because it will always be available when published. Developers can make incremental improvements and enhancements, but the currently delivered application will always be available. The permanent availability of a version of an application means users can choose to opt-in to the new version or not.

Published: May 30, 2022

## Who should read this?

If you are a web developer interested in deploying web applications to the permaweb, you should read this post.

## What do I need to know?

You should know about web development and be familiar with Javascript, NodeJS, and git. You should understand the concepts behind Arweave and the Permaweb. Finally, you should feel comfortable around a command line, and it is not that scary. 😁

## Table of Contents

- Building a web app
- Deploying to the Permaweb
- Setting up a reverse proxy

---

## Building a Web App

More than likely, this step is relatively straightforward for you. You have probably set up many web applications in your day and used several technologies to deploy to the cloud, either using a hosting company or a cloud vendor. Arweave Web3 development is not much different, but there are some little differences you should be aware of because they can be frustrating to troubleshoot if you expect everything to work. And yes, it would be great if everything worked out of the box without thinking about it. And maybe in the future, it will improve. We will create an extremely complicated counter application 😃, and this application will increment and decrement a count, pretty much the hello world of single-page applications. It is also important to note that we are deploying a single-page application. The permaweb supports static web and single-page applications; it does not support server-side rendered applications. There is no dynamic server on the permaweb. 

We will be using Svelte and Vite to create our application. Svelte is a web framework that is small and easy to learn. Vite is a transpiler that can take your code and modules and generate production-ready HTML, CSS, and javascript.

> If you want to follow along, you will need a code editor, and the latest version of Nodejs installed.

### create web app

```
npm create vite dapp -- --template svelte
```

This command should create a new Vite app in the `dapp` directory, so change into that directory `cd dapp` and run the following commands:

```
npm install
npm run dev
```

You should see the Svelte Logo with a counter button, 🎉 congrats, we have a web application in our local environment. We need to set up a few things to handle our deployment on the permaweb. The first thing is to change the base path in the config; when deploying on the Permaweb, we want users to be able to run the app directly from a gateway server. The application must support running in a sub-path. Because our root app path would be `https://{gateway}/{TransactionId}` and all of our assets need to be referenced from the TransactionId path. `https://{gateway}/{transactionId}/index.html` etc. So in the `vite.config.js` file in the `defineConfig` object, we need to add the following property and value. `base: ''`

### setup configuration for permaweb

When setting up your single-page web app to run on the permaweb there are a few configuration changes that must be made from the traditional web server setup. Since the Permaweb is gateway server into the Arweave network, the entry url for your web application is Arweave Transaction Id. This means that your application will run on a sub-path and all of the internal references to your application will need to adjust for this constraint. Typlically references associate by the root-path not a relative path.

``` js
export default defineConfig({
  base: '',
  plugins: [svelte()],
	...
```

Adding the base path to the defineConfig object and setting it to an empty string, we instruct the compiler to configure all references to be relative to the location of the index.html file instead of the root of the webserver.

> NOTE: If you are using Create React App, you would set the `homepage` property in the `package.json` to "."

### install routing component

One of the most powerful features of the web is the ability to share a URL for a specific resource, and if the recipient of that share has the correct permissions, they can click on the link and view the content.

Since single-page applications are dynamic, sharing URLs can result in a 404 because the application dynamically generates the URL. Solutions to this issue are well defined and well documented, and we are going to use the hash-router solution. The hash-router solution adds a hash '#' at the beginning of the path, then any specific route afterwords. The result is now the client side of our application can handle the routing of all links. Tinro is a Svelte routing library that provides these features. https://github.com/AlexxNB/tinro

Install tinro

``` js
npm install -D tinro
```

### create our app

Now that we have our router installed, let's remove the contents of the `src/App.svelte` and add our little app:

``` svelte
<script>
  import { router, Route } from "tinro";
  let count = 0;
  router.mode.hash();
</script>

<Route path="/">
  <h1>Permaweb Counter</h1>
  <p>Count: {count}</p>
  <div>
    <button on:click={() => count++}>Inc</button>
  </div>
  <div>
    <a href="/about">About Page</a>
  </div>
</Route>
<Route path="/about">
  <h1>Deploy to the Permaweb</h1>
  <div>
    <a href="https://github.com/twilson63/deploy-permaweb-demo">github</a>
  </div>
  <div><a href="/">Home Page</a></div>
</Route>

<style>
  div {
    margin-bottom: 16px;
  }
</style>
```

Save that file and `npm run dev`; when you open a web browser to localhost:3000, you should see your new counter and an about page. Let's break down what this new `App.svelte` app does.

In the script tag, we import our `router` and `Route` components, create a local variable `let count = 0`, and tell our router to use the `hash` mode. The hash mode will allow us to take paths from our app and paste the URL into a browser, and you should navigate to the new route.

``` js
import { router, Route } from "tinro";
let count = 0;
router.mode.hash();
```

### index page

Now that we have our core code completed, let's handle our first route:

``` svelte
<Route path="/">
  <h1>Permaweb Counter</h1>
  <p>Count: {count}</p>
  <div>
    <button on:click={() => count++}>Inc</button>
  </div>
  <div>
    <a href="/about">About Page</a>
  </div>
</Route>
```

You can see that the children's contents of the route contain some basic HTML markup and a dynamic insert `{count}` to render the value of the count variable and an expression to handle the `click` event of the button. We increment the `count` variable by 1 using the `++` operator with this click event.

### about page

For the "about" route, we have a simple header and a couple of links, one that points to our Github repo and the other that redirects back to the home page.

``` svelte
<Route path="/about">
  <h1>Deploy to the Permaweb</h1>
  <div>
    <a href="https://github.com/twilson63/deploy-permaweb-demo">github</a>
  </div>
  <div><a href="/">Home Page</a></div>
</Route>
```

Our web app will not win any significant awards, but it works and is ready for the permaweb! 🙌 Great Job!

---

## Deploying to the Permaweb


We have our incredible application, and now we are ready to distribute it to the world so everyone can enjoy our counter application! First, install "arkb," a CLI tool that will create a "path.manifest" document, a router JSON document for the Permaweb. It will upload all of our files and the "path.manifest" to publish our app to the permaweb.

```
npm install -g arkb
```

### What is "arkb"?

"arkb" is a CLI tool that lets you point to a folder on your local system, and "arkb" grabs all the files and uploads them to Arweave, then creates a "path.manifest" of each file mapping out their filename to the Arweave Transaction Identifier on the network. This mapping creates a routing system that Arweave gateways understand how to deliver transactions using web conventions.

https://github.com/textury/arkb

> You may want to setup arlocal and test deploy to a local devnet and make sure everything is working correctly. (see https://github.com/textury/arlocal)

> To test deploy to arlocal
>  `npm install -g arlocal` 
>  npm run build
>  arlocal
>  `arkb deploy dist --gateway http://localhost:1984`


### "arkb" CLI and "bundlr"

There are many ways to use the "arkb" CLI; since all of our files will be less than 256k, we want to use the "Bundlr" feature to post our files to Arweave. What is "Bundlr"? https://bundlr.network/ Bundlr is an L2 technology that allows systems to create batches of data transactions for the Arweave network and post them as a single bundled transaction. The result is faster, cheaper, and more efficient mechanisms to upload data to Arweave. Arweave's minimum data transaction size is 256k, so if you upload 20 bytes or 256k bytes, you will pay the same price for the transaction using the native delivery mechanism. "Bundlr" calculates prices on the actual size of the data (no minimum limit) plus a 30% fee, which results in cheaper data costs. "Bundlr" can inject the data transaction into the gateway cache faster than the native method. Since the effect of a bundled transaction is just a single transaction on Arweave, "Bundlr" can write many transactions in a single block.

Use the`use-bundler` flag to leverage the "bundlr.network" to deploy our application. We also will add a tag name and value to capture the deployment milestone.

```
npm run build
arkb deploy dist --use-bundler node2.bundlr.network --tag-name DEPLOY --tag-value twilson63-counter --auto-confirm
```

The auto-confirm flag will skip the confirmation step, the tag-name and tag-value flags will add a tag name-value pair to each transaction, and finally, the use-bundler flag will publish the app in a bundle. Why? by default, when posting a single transaction to the Arweave network is a minimum of 256k bytes. If you publish several files, no matter how small they are, each file is a separate transaction of 256k; the cost can add up. But with "Bundlr," the transactions are batched together, so the price is the size of the file + 30%, which works out to be relatively small in cost. Many applications may not cost anything to deploy. So use "Bundlr" if the CLI prompts you for a wallet, then check out this "arkb" guide to fund a wallet to deploy.

For more information: https://docs.arweave.org/developers/tools/textury-arkb

### CLI Complete

After about 30 seconds, we should see an Arweave.net URL in our console. Let's use "cmd + click" to open the "link" in a new browser tab, and we should see our incredible application on the permaweb! 🤘 Rock'N'Roll!

---

## Setting up a reverse proxy

Now that our application is on the Permaweb, we may want to use a standard convention to reference and access our application. Domain Naming System "DNS" is the current way to connect a web server to a fully qualified domain name FQDN. Using a reverse proxy as a tool that can consume a "Request" from a browser for a specific domain like 'counter.app' and basically, rename the request location to `https://arweave.net` and make a request to the Arweave gateway. This proxy allows online users to experience an FQDN that they are accustomed to while delivering application data from the Permaweb.

### Connecting a domain name to a permaweb app

There are many ways to handle web traffic, and this is how I choose to do it. I create a reverse proxy. What is a reverse proxy? A reverse proxy will take a target like `https://arweave.net/{txid}` and connect a domain to it like `https://counter.app` so that users that type in `https://counter.app` in their browser will get tunneled through to `https://arweave.net/txid`.

I use a platform called deno.com because they deploy these reverse proxies on edge to about 32 data centers around the world. Go to https://deno.com and create an account. When on the projects page, click on the hello world playground. We will edit this codebase to create our reverse proxy.

``` ts
import { serve } from "https://deno.land/std/http/mod.ts";

const fileService = "https://arweave.net/[ADD YOUR TX ID HERE]";

async function reqHandler(req: Request) {
  const path = new URL(req.url).pathname;
  return await fetch(fileService + path).then(res => {
    // disable when stable
    //return res;

    const headers = new Headers(res.headers)
    headers.set('cache-control', 's-max-age=600, stale-while-revalidate=6000')

     
    
    return new Response(res.body, {
      status: res.status,
      headers
    })
  });
}
serve(reqHandler, { port: 8100 });
```

Replace `[ADD YOUR TX ID HERE]` with your Arweave transaction id. Save and deploy; deno.com will generate a dev URL for you to test this reverse proxy out.

To add your domain to this proxy in the project view, go settings tab and follow the instructions in the domains section. You will set up a DNS record to point to this reverse proxy. Deno.com will generate SSL certificates.


⚡️BAM! We have deployed a project to the permaweb and connected a domain using a reverse proxy to make it easy to access.

---

## Summary

This workshop goes through the process of deploying single-page applications to the Permaweb and it simply covers one of many different ways to get your application on the Permaweb.

### Where can I go if I need help?

The best place to get help is in the [Arweave Developer Talk](https://discord.gg/8vxJCtzBBJ) Discord Channel 
