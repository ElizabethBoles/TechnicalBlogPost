# _*Fetch API Calls*_

### INTRODUCTION

Web developers often need the ability to retrieve data to update a web based application asynchronously. Traditionally, XMLHttpRequest was used to make these requests. The XMLHttpRequest (XHR) object supports a variety of scenarios involving transferring data to and from a server by sending requests. An example of an XMLHttpRequest may look like this:

```function successListener() {  
  let data = JSON.parse(this.responseText);  
  console.log(data);  
}```

```function failureListener(err) {  
  console.log('Request failed', err);  
}```

```let request = new XMLHttpRequest();  
request.onload = successListener;  
request.onerror = failureListener;  
request.open('get', './api/some.json', true);  
request.send();```

In the code above, there are two calls to open() and send() as well as two listeners to handle a successful and failure request. The code above isn't that difficult to read, but as you start to add more and more logic and callbacks, it can quickly become messy which is why XMLHttpRequest needs to be replaced.  

Now, take a look at the code below. The fetch API makes the code above much simpler and easier.

```// url (required), options (optional)
fetch('/some/url', {
	method: 'GET'
}).then(function(response) {
	// success
}).catch(function(err) {
	// something went wrong
});```

The fetch API makes this simpler because it separates the request from the response. It also uses ES6’s new Promise functionality to avoid the hassles of callbacks and events. This is key to fetch APIs because it makes the code more readable and easier to understand.

Let’s start by fetching some data.  The fetch API is easy to use, simply call fetch and provide the url you want to fetch and a parameters you need to provide.

```fetch(url, options);```

Fetch then returns a Promise with a response object that contains additional details such as status code, headers, and other meta data.  

```fetch(url).then(function(response)  {
	console.log(‘Status:’, response.status);
});```

It also provides access to the response data with an understanding of what type of data it is.  


### CHAINING PROMISES
As noted earlier, Promises are integral into the way that fetch APIs function work. For fetch APIs, this means you can share logic between different fetch requests.  Instead of checking the status of a response for an inline function, I can create a separate function to check the status code to return a Promise and get the JSON object from the response stream.

```function checkStatus(response)  {
	if (response.status === 200)  {
	  return Promise.resolve(response);
	} else  {
	  return Promise.reject(
	    new Error(response.statusText));
	}
}```

```function getJSON(response) {  
  	return response.json()  
}```

I can now simplify the fetch request quite a bit. In the sample code below, I will make the fetch request, make sure I get the 200, get the JSON, and then handle the result. If anything goes wrong along the way, I can catch and fix the the error.


```fetch(url)
	.then(checkStatus)
	.then(getJSON)  
  .	then(function(data) {  
	  console.log(‘DATA’, data);  
})
	.catch(function(err) {
	  console.log('ERROR', err);  
});```


### FETCH OPTIONS
In addition to the request, you can have fetch request a set of options.  With fetch options, you can:
	- set the methods to be used
	- add HTTP headers
	- set the Body of the request  
	- add credentials
	- Control how cross domain requests or CORS are handled

### Post
if you wanted fetch to post the results of some user input along with a token, you can use:

```let options = {
  “method”: “post”,
  “headers”:  {
    “Authorization”:  “string XXX”},
  “body”:  “Hello World!”
};```

### COR
Making cross domain requests with fetch uses simpler policies as XMLHttpRequest. It verifies that the correct CORs headers are present before returning the Promise. If the correct headers are not present, the fetch will fail and you will not be able to view the status or the data returned. With fetch, you can also tell the browser how to handle CORs requests with one of 4 fetch mode options:
	- mode:same-origin ensures only requests made to the same origin and everything else is rejected.
	- mode:cors ensure assets on the same organ and on other origins that return the appropriate headers.
	- mode:cors-with-forced-preflight will always perform a preflight check before making the actual request.
	- mode:no-cors which is used to get data from a server that does not support cors but it will result in an opaque response.

Fetch APIs are gaining adaption and support because it simplifies the process to go ahead and make requests. It is fairly straight forward, easy to use and browser support is growing quickly.

### Example

Below is an example of a complete fetch request that we learned at The Iron Yard School from our instructor

(function() {
    'use strict';

    /**
     * Adds the collection of numbers provided together and gives back
     * the result in the promise resolution
     *
     * @param  {Array} numbers The numbers to sum up
     * @return {Promise}       The promise will resolve with the data including the original array and the totalValue
     */
    function sum(numbers) {

        let additionPromise = new Promise(function addPromise( resolve, reject ) {
            if (!Array.isArray(numbers)) {
                // we are telling the promise that the desired action
                // did NOT complete sucessfully
                reject('Hey! Gimme an array of numbers!');
                return;  // we don't want to keep going!!
            }

            let total = 0;
            numbers.forEach(function doAddition(eachNumber) {
                total += eachNumber;
            });

            // I'm ready to inform the Promise that everything went well
            // so we execute the `resolve` function with our resulting data
            resolve( { totalValue: total, numbers: numbers } );
        });

        return additionPromise;
    }

    sum([1, 2, 3])
        .then(function handleTotalValue(theSummingData) {
            console.log('addition complete', theSummingData);
            let newNumbers = [theSummingData.totalValue, 10];

            // returning a Promise from INSIDE a .then() callback
            // will allow you to chain ANOTHER .then() later...
            let aNewPromise = sum(newNumbers);
            return aNewPromise;
        })
        .then(function printFinalResult(moreSummingData) {
            // this SECOND .then() handles the SECOND Promise ("aNewPromise" above)
            // the data will be the result of the SECOND call to sum()
            console.log('more summing data', moreSummingData.totalValue);
            // now a THIRD call to sum() which will REJECT because it is not an array of numbers
            if (moreSummingData.totalValue > 10) {
                return sum('foo');
            }
        })
        .then(function handleThirdSum(theDataFromSumCallNumberThree) {
            console.log('this will not happen');
        })
        .catch(function handleErrors(error) {
            // this catch will catch ANY rejections in the Promise chain
            console.error('we should never get in here!');
        });

    sum('foo')
        // .then() represents the SUCESSFUL case (resolution)
        .then(function didItWork(summingData) {
            console.log('this should not happen!', summingData);
        })
        // we EXPECT this promise to reject (fail), so this callback should execute
        .catch(function handleErrors(error) {
            console.error(error);
            console.info('this is an info message');
            console.warn('this is a warning');
        });


    /**
     * Retrieves data from GitHub for a user
     * @return {Promise} The promise from the API call to GitHub with the data
     */
    function getData() {
        // this whole thing, returns a Promise... the one returned by response.json()
        return fetch('https://api.github.com/users/jakerella')
            .then(function handleResponse(response) {
                if (response.status > 199 && response.status < 300) {
                    return response.json();
                } else {
                    // creating a new promise and immediately rejecting it
                    return Promise.reject('Looks like a bad status code');
                }
            });
    }

    /**
     * Logs out the result of getting the data
     *
     * @return {void}
     */
    function putDataInThePage() {
        getData()
            // this .then() is handling the SECOND promise above...
            // the one returned by response.json()
            .then(function handleTheRsponseData(userData) {
                console.log(userData);
            })
            .catch(function handleErr(err) {
                console.error(err);
            });
    }

    putDataInThePage();

})();
