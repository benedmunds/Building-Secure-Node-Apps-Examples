# Chapter 4 - Authentication, Access Control, and Safe File Handing

## Authentication

```
var bcrypt = require('bcrypt');

//this should be a separate module
//in a real world system
var auth = {

	//start with a default state of failure
	user: { authenticated: false },
	message: '',

	login: function(password, callback, errCallback){

		var _this = this;

		//grab the password hash from our imaginary db
		var passwordHash = db.user.passwordHash;

		//now check to see if the login password matches
		bcrypt.compare(
			password,
			passwordHash,
			function(err, res){

				if (res === true) {
					_this.user = db.user;
					_this.user.authenticated = true;
					_this.message = 'Logged in';

					callback(_this);
				}

				_this.message =
				  'Incorrect username or password';

				errCallback(_this);

		});

	}

};


//login the user
auth.login(req.body.password, function(res){

	//successfully authenticated
	req.user = res.user;

}, function(res){

	var errorMsg = res.message;

});
```


## Access Control

### Authenticate User
```
//define the middleware function
function isAuthenticated(req, res, next) {

	//determine if we have an authenticated user
	if (typeof(req.user) !== 'undefined' &&
	    req.user.authenticated === true) {

	    return next();

	}


	//the user is not authenticated
	//redirect them to login
	res.redirect('/login');

}


//add the middle to our /user routes
app.all(
	'/user/*',
	isAuthenticated,
	function(req, res) {

  //will only get here if
  //the user is authenticated

});
```

### Authenticate Admin
```
//define the isAuthenticated middleware function
function isAuthenticated(req, res, next) {

	//determine if we have an authenticated user
	if (typeof(req.user) !== 'undefined' &&
	    req.user.authenticated === true) {

	    return next();

	}


	//the user is not authenticated
	//redirect them to login
	res.redirect('/login');

}

//define the isAdmin middleware function
function isAdmin(req, res, next) {

	//determine if we have an admin user
	if (typeof(req.user) !== 'undefined' &&
	    req.user.groups.indexOf('admin') !== -1) {

	    return next();

	}


	//the user is not authorized
	//redirect them back home
	res.redirect('/');

}


//add the middle to our /user routes
app.all(
  '/user/*',
  isAuthenticated,
  isAdmin,
  function(req, res) {

		//will only get here if the user
		//is authenticated as an admin

});
```

### Routes
```
//will allow all users to get
app.get('/blog', isAuthenticated, function(req, res){});

//must be an editor to post
app.post('/blog', isAuthenticated, isEditor, function(req, res){});

//must be an editor to put
app.put('/blog', isAuthenticated, isEditor, function(req, res){});

//must be an admin to delete
app.delete('/blog', isAuthenticated, isAdmin, function(req, res){});
```

## Validating Redirects

### Redirect
```
if (valid === true && dataSaved === true) {
	res.redirect('/blog/1/edit');
}
```

### Direct Call
```
if (valid === true && dataSaved === true) {
	app.edit(1);
}
```

### Middleware
```
if (valid === true && dataSaved === true) {
	res.redirect('/blog/1/edit');
}


//...


//define the middleware function
function verifyAdditionalData(req, res, next) {

	//verify additional data/credentials/whatever
	if (valid === true && dataSaved === true) {

		return next();

	}


	//the user is not authorized
	//redirect them back to the blog
	res.redirect('/blog/' + req.params.blogId);

}


//blog edit routes
app.all(
  '/blog/:blogId/edit',
  isAuthenticated,
  isEditor,
  verifyAdditionalData,
  function(req, res){

  //...

});
```


## Obfuscation

```
var Hashids = require('hashids');
var hashids = new Hashids('App-wide salt');

app.all(
  '/blog/:blogHash/edit',
  isAuthenticated,
  isEditor,
  function(req, res) {

		//blogHash = BaPjae
		var id = hashids.decode(req.params.blogHash);

		var post = db.post().where({

		  'id': id

		}).row();

});
```


## Safe File Handing
```
//define the accounting middleware function
function isAccountant(req, res, next) {

	//determine if we have an accountant user
	if (typeof(req.user) !== 'undefined' &&
	  req.user.groups.indexOf('accountant') !== -1) {

		return next();

	}


	//the user is not authorized
	//redirect them back home
	res.redirect('/');

}


//blog edit routes
app.get(
  '/accounting/statements/:year/:month',
  isAuthenticated,
  isAccountant,
  function(req, res) {

		//the user has access

		//set our file options and headers
		var options = {
			root: __dirname + '../uploads/acct/stmnts/',
			dotfiles: 'deny',
			headers: {
			  'Content-type': 'application/pdf'
			  'Expires': '0',
			  'Cache-Control': 'must-revalidate',
			  'Pragma': 'public'
			}
		};

		//define filename
		var filename  = parseInt(req.params.year) + parseInt(req.params.month) + '.pdf';


		//send the file contents to the browser
		res.sendFile(filename, options, function(err) {

			if (err) res.status(err.status).end();

			//file served successfully

		}).end();

});
```
