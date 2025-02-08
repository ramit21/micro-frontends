# Micro frontends POC

Theory: https://www.linkedin.com/posts/ramit-sharma-48110060_activity-7004091756652752896-T7pB

Another microfrontend POC with routing explained: https://github.com/ramit21/react-microfrontends

## Setup:
Run 'npm install' and then 'npm start' (which spins up webpack server) inside all 3 folders: container (aka **shell app**), products, cart. Open http://localhost:8081/ for product, http://localhost:8082/ for cart, http://localhost:8080/ for the main container app.

## Container and Remote app setup
See the webpack.config.js on how files are exposed from remote apps, and included in container app to be further imported for use.

index.js and bootstrap.js are kept separate to allow for asyncrhonous loading so that webpack gets the opportunity to load the modules imported from other components. 

Index.html of the remote apps is only for rendering the application on rage for development. Index.html of the container app on the other hand is for both rage development, as well as production deployed application which points to and calls the remote applications.

If you try to render same html component by id in both container app, and the remote app, it may cause issue if the HTML element of same id does not exist in both remote and the container app HTMLs. The work around is to refactor the code as done in product->src/bootstrap.js -> mount method, so that the content is loaded to the correct HTML element as per the environment (development or production). Also see bootstrap of container application on how the mount method is called to render the HTML element.

## How webpack works?
Webpack combines many js files into a single bundle.js (or main.js) file. It starts with index.js, traverses the tree of dependencies, and combines each into the single bundle.js file. If you open the bundle.js, you will see many long eval statement. The code inside each eval is actually the file that was loaded by the webpack as per depenency tree. To expose this bundle.js/main.js to the browser, use the webpack server, the configs for which are given in webpack.config.js

The bundle.js, or aka main.js created by webpack for our remote micro-frontend app, needs to be added to the container app html, like a script tag include. This function is achieved by using **WebpackFederationModule**. Use this plugin and choose files to expose from the remote (micro-frontend) applications, and use it in container app to choose what files to include from different remotes. This plugin generates more output files than just a single bundle.js. When used with remote app, the plugin generates a ‘remoteEntry.js’ file that acts like a manifest file pointing to information on what and how to load other files like src_index.js, someDependency.js etc. These files are basically browser loadable versions of their respective components in the remote app. Coming to container app, first thing you need to do is to load your content asynchronously so that FederationModule gets time to evaluate imports in the code which point to exposed files of remote apps. The imports to remote app files are then checked with remoteEntry.js, which then points to the files that needs to be loaded in the browser.

## Additional theory

1. How do you handle authentication and authorization in a microfrontend setup? Ans. On user login, maintain user's JWT token in browser's localstorage, or in common Redux store, or pass it as cookes, or parent container props from container to remote apps. eg:
```
 return (
    <div>
      <h1>Container App</h1>
      <RemoteApp jwt={userJwt} />
    </div>
  );
```
2. Describe how you would implement communication between two microfrontends. Ans. Using central data store like Redux/Zustand, or by using CustomEvents eg:
```
const event = new CustomEvent('updateData', { detail: { data: 'Some data' } });
window.dispatchEvent(event);

// Microfrontend B
window.addEventListener('updateData', (event) => {
console.log(event.detail.data); // Outputs: 'Some data'
```
3. What strategies can be used for code splitting in microfrontends? Ans. Based on routes to different features. You may also split based on components, with heavy not-frequently components loaded lazily as a seperate mircorftontend.
4. How do you ensure consistent styling across microfrontends? Ans. Use common CSS libraries for consistent styling. Use CSS-in-JS library to contraint remote app css to themselves and not impcat styling on other apps.
5. What are the best practices for testing microfrontends? ans. Unit test all MFEs in isolation. PACT contract tests for API integrations. Automation test suites like using Cypres to test from container app in a deployed env. Finally load test your application.
6. 
