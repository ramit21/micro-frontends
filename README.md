# micro-frontends
Micro frontends POC

Steps to run the POC:
Run 'npm install' and then 'npm start' (which spins up webpack server) inside all 3 folders: container, products, cart. Open http://localhost:8081/ for product, http://localhost:8082/ for cart, http://localhost:8080/ for the main container app.

See the webpack.config.js on how files are exposed from remote apps, and included in container app to be further imported for use.

index.js and bootstrap.js are kept separate to allow for asyncrhonous loading so taht webpack gets the opportunity to load the modules imported from other components. 

Index.html of the remote apps is only for rendering the application on local for development. Index.html of the container app on the other hand is for both local development, as well as production deployed application which points to and calls the remote applications.

If you try to render same html component by id in both container app, and the remote app, it may cause issue if the HTML element of same id does not exist in both remote and the container app HTMLs. The work around is to refactor the code as done in product->src/bootstrap.js -> mount method, so that the content is loaded to the correct HTML element as per the environment (development or production). Also see bootstrap of container application on how the mount method is called to render the HTML element.


