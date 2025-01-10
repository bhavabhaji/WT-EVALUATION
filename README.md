1)	Helloworld.js
var http=require('http');
http.createServer(function(req,res) {
res.writeHead(200,{'Content-Type':'text/html'});
res.write("<body bgcolor='orange'>");
res.write("Hello World");
res.write("</body");
res.end("<h1> good morning</h1>");
console.log("conneceting");
}).listen(8080);


2)	Gcd.js
var url=require('url');
var http=require('http');
function gcd(a, b) 
{
    if (b === 0)
    {
        return a;
    }
    else
          return gcd(b, a % b);

}
http.createServer(function(req,res) {
res.writeHead(200,{'Content-Type':'text/html'});
var q=url.parse(req.url,true).query;

var a=q.a;
var b=q.b;
a=parseInt(a);
b=parseInt(b);
res.write("the gcd of two numbers is" + gcd(a,b)+"<br>");

res.end();

}).listen(8000);

3)	Database.js
var mysql= require('mysql');
var con=mysql.createConnection({
host:"127.0.0.1",
user:"root",
password: "",

});

con.connect(function(err) {
  if(err) throw err;
  console.log("Connected");
  con.query("CREATE DATABASE Varghese",
      function(err,result){
        if (err) throw err;
        console.log("Database Created");
 });

});

4)	Table.js
var mysql= require('mysql');
var con=mysql.createConnection({
host:"127.0.0.1",
user:"root",
password: "",
database:"Varghese"
});

con.connect(function(err) {
  if(err) throw err;
  console.log("Connected");
  var sql="CREATE TABLE details(custname varchar(5), city varchar(30),salesperson varchar(30))";
  con.query(sql,
      function(err,result){
        if (err) throw err;
        console.log("TABLE Created");
 });

});


5)	Crud operations
var http=require('http');
var sql=require('mysql');
var express=require('express');
var app=express();
var bodyParser=require('body-parser');
var urlencodedParser=bodyParser.urlencoded({
    extended:true
});
 var con= sql.createConnection({
    host:"127.0.0.1",
    user:"root",
    password:"",
    database:"Varghese"
 });

 app.get('/send',function(req,res)
 {
      var rr="<html>";
      rr=rr+"<body>";
      rr=rr+"<form method='POST' action='thank'>";
      rr=rr+"Customername"+"<input type='text' name='one' value=' '>";
      rr=rr+"City"+"<input type='text' name='two' value=' '>";
	  rr=rr+"Salesperson"+"<input type='text' name='three' value=' '>";
      rr=rr+"<input type='submit' name='t1' value='send'>";
	  rr=rr+"</form>";
	   //update records form
	  rr=rr+"<h3>Update Record</h3>"
      rr=rr+ "<form method='POST' action='update'>";
      rr=rr+"Existing Customer Name: <input type='text' name='existingName' value=' ' required><br>";
      rr=rr+"New City: <input type='text' name='newCity' value=' ' required><br>";
      rr=rr+"New Salesperson: <input type='text' name='newSalesperson' value=' ' required><br>";
      rr=rr+"<input type='submit' name='t2' value='Update'>";
      rr=rr+"</form>"
	  //delete form
	  rr=rr+"<h3>Delete Record</h3>"
      rr=rr+"<form method='POST' action='delete'>";
      rr=rr+"Customer Name to Delete: <input type='text' name='deleteName' value=' ' required><br>";
      rr=rr+"<input type='submit' name='t3' value='Delete'><br>";
      rr=rr+"</form>";  
	   //view records
	  rr=rr+"<h3>View Records</h3>"
      rr=rr+"<form method='GET' action='display'>";
      rr=rr+"<input type='submit' name='t4' value='View All Records'>";
      rr=rr+"</form>";
      rr=rr+"</body>";
      res.send(rr);
     
      
 }
 );

 app.post('/thank', urlencodedParser,function(req,res)
 {
     var reply='';
     custname=req.body.one;
     city=req.body.two;
     salesperson=req.body.three;
     var sql="insert into details(custname,city,salesperson) values('"+custname+"','"+city+"','"+salesperson+"')";

 con.connect(function(err)
    {
if(err) throw err;
console.log("Connected");

});
   con.query(sql,function(err,result)
   {
     if(err) throw err;
     res.write("record inserted");

 
   });

  res.write("Record inserted");
  res.end();

});

//updating record
app.post('/update', urlencodedParser, function (req, res) {
    var existingName = req.body.existingName;
    var newCity = req.body.newCity;
    var newSalesperson = req.body.newSalesperson;

 var sql = "UPDATE details SET city = ?, salesperson = ? WHERE custname = ?";
    con.query(sql, [newCity, newSalesperson, existingName], function (err, result) {
        if (err) {
            res.status(500).send("Error updating record: " + err.message);
        } else if (result.affectedRows === 0) {
            res.send("No record found with the given name.");
        } else {
            res.send("Record updated successfully!");
        }
    });
});

//delete record
app.post('/delete', urlencodedParser, function (req, res) {
    const deleteName = req.body.deleteName;

  const sql = "DELETE FROM details WHERE custname = ?";
    con.query(sql, [deleteName], function (err, result) {
        if (err) {
            res.status(500).send("Error deleting record: " + err.message);
        } else if (result.affectedRows === 0) {
            res.send("No record found with the given name.");
        } else {
            res.send("Record deleted successfully!");
        }
    });
});


app.get('/display', function (req, res) {
    const sql = "SELECT * FROM details";
    con.query(sql, function (err, results) {
        if (err) {
            res.status(500).send("Error retrieving records: " + err.message);
        } else {
            var displayHTML = `
<html>
                <body>
                    <h3>All Records</h3>
                    <table border="1">
                        <tr>
                            <th>Customer Name</th>
                            <th>City</th>
                            <th>Salesperson</th>
                        </tr>`;

results.forEach(record => {
                displayHTML += `
			<tr>
                            <td>${record.custname}</td>
                            <td>${record.city}</td>
                            <td>${record.salesperson}</td>
                        </tr>`;
            });

            displayHTML += `
                    </table>
                    <br>
                    <a href="/send">Back to Home</a>
                </body>
            </html>`;
            res.send(displayHTML);
        }
    });
});

// Start the server
app.listen(9000, () => {
    console.log('Server is running on http://localhost:9000');
});

