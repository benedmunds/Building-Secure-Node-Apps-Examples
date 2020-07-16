# Chapter 1 

## How to Guard Against It

```
var pg = require('pg');
var client = new pg.Client();

// connect to our database
client.connect(function (err) {
  if (err) throw err;

  // execute a query on our database
  client.query(
  'UPDATE users SET first_name = $1::text WHERE id = $2::int',
    [req.body.first_name, 1001],
    function (err, result) {

    	//...

  });
});
```


## Mass Assignment

### Whitelist
```
var whitelist = ['first_name', 'last_name', 'email'];
var data = {};

for (var property in req.body) {
    if (
      req.body.hasOwnProperty(property) &&
      whitelist.indexOf(property) !== -1) {

        data[property] = req.body[property];

    }
}

var User = mongoose.model('User');

var liz = new User(data);
liz.save();
```


## Typecasting
```
var id = 1001;
var first_name = req.body.first_name;

connection.query(
  'UPDATE users SET first_name = ?
   WHERE id = ?',

  [first_name, Number(id)],

  function(err, result) {

      //...

	});
```


## Sanitizing Output

### Outputting to the Browser
```
var escape = require('html-escape');

var badData = "
     <script>
       alert('I am not sanitized!');
     </script>
";

var goodData = escape(badData);
```


### Echoing to the Command Line
```
var shellescape = require('shell-escape');

var args = [
  'curl', '-v', '-H',
  'Location;', '-H',
  'User-Agent: testing#123',
  'http://buildsecurenodeapps.com/'
];

var escaped = shellescape(args);
console.log(escaped);
```