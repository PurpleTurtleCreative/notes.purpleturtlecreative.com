---
title: API Web Worker
parent: JavaScript
nav_order: 1
---

# API Web Worker

A simple Web Worker script to thread API fetch requests.

See [*Using Web Workers*](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) on MDN for details on using JavaScript Web Workers in general.

*api-worker.js*

```javascript
/**
 * Worker to fetch data from an API endpoint.
 *
 * @param method The HTTP request method.
 * @param route The URL to request.
 * @param body Optional. Object for request body data.
 */
onmessage = ({ data }) => {

	const init = {
		"method": data.method
	}

	if ( data.body ) {
		init.data = new URLSearchParams( data.body );
	}

	fetch( data.route, init )
		.then( res => res.json() )
		.then( data => postMessage( data ) );
}
```

*main.js*

```javascript
const worker = new Worker('/api-worker.js');
worker.onmessage = (e) => {
	finalizeWorker( worker, e );
}

const queryString = new URLSearchParams({
	"active": true,
	"limit": 100
}).toString();

worker.postMessage({
	"method": 'GET',
	"route": `/api/test?${queryString}`
	"body": {
		"search": 'Example',
		"user_id": 123
	}
});

/**
 * Processes a worker's message and terminates the worker.
 *
 * @param Worker worker
 * @param MessageEvent event
 */
function finalizeWorker( worker, event ) {

	// Terminate worker.
	worker.terminate();

	// Process response.
	const res = event.data;
	if ( 'success' === res.status ) {
		if ( res.results && res.results.length > 0 ) {
			console.log('Results were received from Web Worker\'s request!');
		}
	} else {
		console.error('Failed /api/test request.');
	}
}
```

