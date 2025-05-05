# Frontend-caching



This article by Brock Reece provides a concise overview of frontend caching strategies, emphasizing the importance of optimizing data retrieval in web applications. Here’s a breakdown of the key strategies discussed, along with a suggestion to incorporate both `fetch` and `axios` in examples for broader applicability.

### Frontend Caching Strategies

1. **No Caching**: 
   - This is the simplest approach where data is fetched directly from the API without any caching mechanism.
   ```javascript
   let data;
   axios.get('/data').then((response) => {
     data = response.data.data;
   });
   ```
   - While straightforward, this method can lead to increased latency, affecting user experience.

2. **Prefer API**: 
   - Here, the last successful response is cached and used if the network request fails.
   ```javascript
   let data;
   axios.get('/data').then((response) => {
     data = response.data.data;
   }).catch(() => {
     data = localStorage.getItem('data');
   });
   ```
   - This strategy enhances reliability but offers limited performance improvements.

3. **Prefer Local**: 
   - In this strategy, data is retrieved from local storage if available; otherwise, a network request is made.
   ```javascript
   let data = localStorage.getItem('data');
   if (!data) {
     axios.get('/data').then((response) => {
       data = response.data.data;
       localStorage.setItem('data', data);
     });
   }
   ```
   - This can significantly improve performance for static data.

4. **Local then API**: 
   - This hybrid approach retrieves data from local storage first and then updates it via a network request.
   ```javascript
   const currentVersion = 2;
   let data = localStorage.getItem('data');
   let version = localStorage.getItem('version');
   if (!version || version < currentVersion) {
     axios.get('/data').then((response) => {
       data = response.data.data;
       localStorage.setItem('data', data);
       localStorage.setItem('version', currentVersion);
     });
   }
   ```
   - This method balances performance and data freshness, making it a robust choice.

### Summary
Implementing a well-designed caching strategy can significantly enhance user experience and reduce server load. Depending on the specific requirements of your application, a combination of these strategies might be the best approach.

### Additional Considerations
For those interested in more advanced caching techniques, exploring Service Workers for fine-grained network control and using push-based technologies like WebSockets for real-time data updates could be beneficial.

### Example with Fetch
To provide a broader perspective, here’s how you could implement a similar strategy using the `fetch` API:

1. **No Caching**:
   ```javascript
   fetch('/data')
     .then(response => response.json())
     .then(data => {
       console.log(data);
     });
   ```

2. **Prefer API**:
   ```javascript
   fetch('/data')
     .then(response => response.json())
     .then(data => {
       console.log(data);
     })
     .catch(() => {
       const cachedData = localStorage.getItem('data');
       console.log(cachedData);
     });
   ```

3. **Prefer Local**:
   ```javascript
   let data = localStorage.getItem('data');
   if (!data) {
     fetch('/data')
       .then(response => response.json())
       .then(data => {
         localStorage.setItem('data', JSON.stringify(data));
       });
   }
   ```

4. **Local then API**:
   ```javascript
   const currentVersion = 2;
   let data = localStorage.getItem('data');
   let version = localStorage.getItem('version');
   if (!version || version < currentVersion) {
     fetch('/data')
       .then(response => response.json())
       .then(data => {
         localStorage.setItem('data', JSON.stringify(data));
         localStorage.setItem('version', currentVersion);
       });
   }
   ```

By using both `axios` and `fetch`, developers can choose the best tool for their specific needs while applying similar caching strategies.



The `cache` property of the `Request` interface in the Fetch API allows developers to control how HTTP requests interact with the browser's cache. Understanding the different cache modes can help optimize performance and resource management in web applications. Here’s a breakdown of the available values for the `cache` property, along with examples for each:

### Cache Modes

1. **default**:
   - The browser checks the cache for a matching request.
   - If found and fresh, it returns the cached response. If stale, it makes a conditional request to the server.
   - If no match, it performs a normal request and updates the cache.
   ```javascript
   fetch("some.json", { cache: "default" }).then(response => {
     /* consume the response */
   });
   ```

2. **no-store**:
   - The browser fetches the resource from the server without checking the cache and does not update the cache.
   ```javascript
   fetch("some.json", { cache: "no-store" }).then(response => {
     /* consume the response */
   });
   ```

3. **reload**:
   - The browser fetches the resource from the server without checking the cache but updates the cache with the downloaded resource.
   ```javascript
   fetch("some.json", { cache: "reload" }).then(response => {
     /* consume the response */
   });
   ```

4. **no-cache**:
   - The browser checks the cache for a matching request. If found, it makes a conditional request to the server to verify freshness.
   - If no match, it performs a normal request and updates the cache.
   ```javascript
   fetch("some.json", { cache: "no-cache" }).then(response => {
     /* consume the response */
   });
   ```

5. **force-cache**:
   - The browser returns a cached response if available, regardless of freshness. If no match is found, it makes a normal request and updates the cache.
   ```javascript
   fetch("some.json", { cache: "force-cache" }).then(response => {
     /* consume the response */
   });
   ```

6. **only-if-cached**:
   - The browser looks for a matching request in the cache. If found, it returns it; if not, it responds with a 504 Gateway timeout.
   - This mode can only be used with the `same-origin` request mode.
   ```javascript
   let controller = new AbortController();
   fetch("some.json", {
     cache: "only-if-cached",
     mode: "same-origin",
     signal: controller.signal,
   }).catch(e => {
     if (e instanceof TypeError && e.message === "Failed to fetch") {
       return { status: 504 }; // Workaround for Chrome's TypeError
     }
     return Promise.reject(e);
   }).then(res => {
     if (res.status === 504) {
       // Handle 504 status
     }
     /* consume the response */
   });
   ```

### Summary
Using the appropriate cache mode can significantly improve the performance of web applications by reducing unnecessary network requests and managing resource loading effectively. Depending on the context, developers can choose the best strategy to balance between freshness and performance. 

### Browser Compatibility
The `cache` property is supported in most modern browsers, but it's always good to check for compatibility based on your target audience. For instance, the `only-if-cached` mode is supported in Chrome, Firefox, and Edge, but not in Safari.

### Additional Considerations
When implementing caching strategies, consider the nature of the data being fetched and how often it changes. This will help you choose the right caching mode to optimize performance while ensuring data accuracy.
