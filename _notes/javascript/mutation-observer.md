---
title: Mutation Observer
parent: JavaScript
nav_order: 1
---

# Mutation Observer

Cloning a pagination element when the container updates (ie. the page changes and the list of items updates).

*See [MutationObserver documentation](https://developer.mozilla.org/en-US/docs/Web/API/MutationObserver) on MDN.*

```javascript
document.addEventListener('DOMContentLoaded', function() {

	const observerTarget = document.getElementById("learndash-content");
	const observerOpts = { childList: true, subtree: true };

	// Initially prepend the pagination node.
	const initialPagination = observerTarget.querySelector(".bb-lms-pagination");
	initialPagination.parentElement.prepend(initialPagination.cloneNode(true));

	let observer = new MutationObserver(function (m) {
		m.forEach(element => {
			element.addedNodes.forEach(node => {
				if ( node && node.classList && node.classList.contains("bb-lms-pagination") ) {
					observer.disconnect(); // prevent infinite loop.
					node.parentElement.prepend(node.cloneNode(true));
					observer.observe(observerTarget, observerOpts); // reconnect.
				}
			});
		});
	});

	// Prepend the pagination node every time the DOM changes.
	observer.observe(observerTarget, observerOpts);
});
```
