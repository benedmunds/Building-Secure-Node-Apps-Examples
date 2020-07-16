# Chapter 5 - Safe Defaults, Cross Site Scripting, and Other Popular Hacks

## Never Trust Yourself - Use Safe Defaults
```
app.post('/signup', function() {

  //the default logic is failure
  var message = 'Invalid Form Data';


  //check for a valid form
  if (form.validate('signup') === true) {

		//process the form
		var message = 'Form Valid';

  }

});
```

## Don't Trust Dynamic Typing

### Dynamic
```
//this usually happens when pulling data
//from a data store that doesn't support
//the native boolean type
var isActive = 'false';

if (isActive) {

	console.log('we shouldnt be here, yet we are');

}
else {

	console.log('we should be here');

}

// output
// we shouldnt be here, yet we are
```

### Explicit
```
if (isActive === true) {

	console.log('we shouldnt be here, yet we are');

}
else {

	console.log('we should be here');

}

// output
// we should be here
```


## Cross Site Request Forgery

### Token Generation
```
function generateCsrf(req, res, next) {

	//safe default!
	req.session.csrfToken = null;

	//generate a new token
	var crypto = require('crypto');
	crypto.randomBytes(48, function(err, buffer) {

	  var token = buffer.toString('hex');

	  //save the token in the session and proceed
	  req.session.csrfToken = token;

	  next();

	});

};
```

### Token Verification - Render
```
app.get(
	'/signup',
	generateCsrf,
	function(req, res) {

		res.render('signup/form', {
			csrfToken: req.session.csrfToken
		});

});
```

### Token Verification - View
```
<form method="POST" action="/signup">

  <label>
	First Name:
	<input type="text" name="first_name" />
  </label>

  <label>
	Last Name:
	<input type="text" name="last_name" />
  </label>

  <label>
	Email:
	<input type="text" name="email" />
  </label>

  <input type="hidden" name="token" value="{{csrfToken}}" />

  <input type="submit" name="submit" value="Signup" />

</form>
```

### Token Verification - Validate
```
function validCsrf(req, res, next) {

	if (req.body.csrfToken === req.session.csrfToken) {

		//clear token since it's been consumed
		req.session.csrfToken = null;

		next();

	}

};

app.post('/signup', validCsrf, function(req, res) {
{

	//process data

});
```


## Race Conditions

### Token Generation - Fail
```
function generateCsrf(req, res, next) {

	//safe default!
	req.session.csrfToken = null;

	//generate a new token
	var crypto = require('crypto');
	crypto.randomBytes(48, function(err, buffer) {

		//this will run after the below
		//code has been ran
		var token = buffer.toString('hex');

	});


	//save the token in the session and proceed
	req.session.csrfToken = token;

	next();

};
```

### Token Generation - Fixed
```
function generateCsrf(req, res, next) {

	//safe default!
	req.session.csrfToken = null;

	//generate a new token
	var crypto = require('crypto');
	crypto.randomBytes(48, function(err, buffer) {

		var token = buffer.toString('hex');

		//save the token in the session and proceed
		req.session.csrfToken = token;

		next();

	});

};
```
