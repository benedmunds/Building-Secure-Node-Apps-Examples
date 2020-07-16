# Chapter 3 - Password Encryption and Storage for Everyone

## Putting It All Together

### Hashing with Salt
```
var bcrypt = require('bcrypt');

var saltRounds = 10;

//the plaintext password submitted via a form
var password = req.body.password;

bcrypt.genSalt(saltRounds, function(err, salt) {

  bcrypt.hash(
    password,
    salt,
    function(err, passwordHash) {

	    //store the hash in your user datastore

  });

});
```



### Hashing with Automatic Salt
```
var bcrypt = require('bcrypt');

var saltRounds = 10;

//the plaintext password submitted via a form
var password = req.body.password;

bcrypt.hash(
  password,
  saltRounds,
  function(err, passwordHash){

	  //store the hash in your user datastore

});
```

### Password Verification
```
var bcrypt = require('bcrypt');

//start with a default state of failure
//always assume failure first
var valid = false;

//the plaintext password submitted via a form
var password = req.body.password;

//grab the password hash from our imaginary db
var passwordHash = db.user.passwordHash;


//now check to see if the login password matches
bcrypt.compare(
  password,
  passwordHash,
  function(err, res){

		if (res === true) {
			valid = true;
		}


		if (valid === true) {
		  //valid auth stuff
		}

});
```


### Hashing using Promises
```
var bcrypt = require('bcrypt');

var saltRounds = 10;
var password = req.body.password;

bcrypt.hash(password, saltRounds)
      .then(function(passwordHash) {

	//store the hash in your user datastore

});
```


### Password Verification using Promises
```
var bcrypt = require('bcrypt');

//start with a default state of failure
//always assume failure first
var valid = false;

//the plaintext password submitted via a form
var password = req.body.password;

//grab the password hash from our imaginary db
var passwordHash = db.user.passwordHash;

//now check to see if the login password matches
bcrypt.compare(password, passwordHash)
      .then(function(res){

	if (res === true) {
		valid = true;
	}


	if (valid === true) {
	  //valid auth stuff
	}

});
```



## Upgrading Legacy Systems

### Upgrade Path 1
```
var md5 = require('md5');
var bcrypt = require('bcrypt');

//start with a default state of failure
//always assume failure first
var valid = false;

var saltRounds = 10;

//the plaintext password submitted via a form
var password = req.body.password;

//grab the password hash from our imaginary db
var passwordHash = db.user.passwordHash;

//grab the password algo from our imaginary db
//when you create this column you'll default it to
//the algo you used previously
var passwordAlgo = db.user.passwordAlgo;


if (passwordAlgo === 'bcrypt') {

	//they have previously logged in and
	//upgraded their hash so they're good
	//proceed with verifying login as usual
    bcrypt.compare(
      password,
      passwordHash,
      function(err, res){

				if (res === true) {
				  valid = true;
				}


				if (valid === true) {
				  //valid auth stuff
				}

    });

}
else {

	//this is modified from our old hash check
	//your legacy code will definitely differ
	var oldHash = md5(req.body.password);

	if (oldHash === passwordHash) {

		//generate the new hash with the
		//plaintext password they just gave us
		bcrypt.hash(
		  password,
		  saltRounds,
		  function(err, passwordHash){

				valid = true;

				var passwordAlgo = 'bcrypt';

				//save the new passwordHash and
				//passwordAlgo to the database

		});

	}

}
```


### Upgrade Path 2 - Step 1 - Script
```
var bcrypt = require('bcrypt');

var saltRounds = 10;

//grab all our users from the database
var users = db.users;

users.forEach(function(user){

	bcrypt.hash(
	  password,
	  saltRounds,
	  function(err, newHash){

		  //save newHash to the database

  });

});
```

### Upgrade Path 2 - Step 2 - Registration
```
var md5 = require('md5');
var bcrypt = require('bcrypt');

var saltRounds = 10;

//hash it with MD5
var intermediatePasswordHash = md5(req.body.password);

//then hash that with BCrypt
bcrypt.hash(
  intermediatePasswordHash,
  saltRounds,
  function(err, passwordHash){

	  //save the passwordHash to the database

});
```

### Upgrade Path 2 - Step 2 - Login
```
var md5 = require('md5');
var bcrypt = require('bcrypt');

//start with a default state of failure
//always assume failure first
var valid = false;

var saltRounds = 10;

//the plaintext password submitted via a form
var password = req.body.password;

//grab the password hash from our imaginary db
var passwordHash = db.user.passwordHash;

//md5 hash the password before we bcrypt it
var intermediatePasswordHash = md5(password);

bcrypt.compare(
  intermediatePasswordHash,
  passwordHash,
  function(err, res){

	if (res === true) {
	  valid = true;
	}


	if (valid === true) {
	  //valid auth stuff
	}

});
```
