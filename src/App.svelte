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
