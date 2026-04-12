# Shane Colledge  
## Computer Science Portfolio  

Welcome to my professional ePortfolio. This portfolio demonstrates my growth and technical expertise in:

- Full-stack application development  
- Software engineering and system architecture  
- Algorithmic optimization and data structures  
- Database design and engineering  
- Secure software development practices  

---

## Capstone Overview  

This portfolio is part of my CS 499 Computer Science Capstone. It includes:

- A narrated code review  
- Enhanced full-stack application artifacts  
- Technical enhancement narratives aligned with program outcomes  
- A professional self-assessment  

---

## Featured Artifact  

**Travlr Getaways – Full-Stack MEAN Application**

A full-stack web application built using Angular, Express, Node.js, and MongoDB.  
Enhancements will focus on:

- Architectural refinement  
- Algorithmic efficiency improvements  
- Database optimization and indexing  
- Security hardening and validation controls

- **Project Code:**
https://github.com/hiphoss/CS-465

---

## Contact  

GitHub: https://github.com/hiphoss  

---

## Code Review

My code review of the Travlr Getaways full-stack application can be viewed here:

[Watch Code Review Video](https://youtu.be/-LIaatrDvtc)

---

## Enhancement One: Software Design and Engineering

This enhancement focuses on improving error handling in the Travlr Getaways full-stack application.

The original version of the application had limited and inconsistent error handling, which made debugging more difficult. To improve this, I implemented centralized error handling middleware. This allows the application to manage errors in a consistent way and improves overall system reliability.

This enhancement demonstrates better software design practices by improving maintainability and making the application easier to extend. It also supports a more structured approach to handling unexpected issues within the system.

**Key Improvements:**
- Centralized error handling middleware
- Consistent API error responses
- Improved maintainability and structure
  
var createError = require('http-errors');
var express = require('express');
var path = require('path');
var cookieParser = require('cookie-parser');
var logger = require('morgan');
var hbs = require('hbs');
var errorHandler = require('./errorHandler');

var indexRouter = require('./app_server/routes/index');


var app = express();

// view engine setup
app.set('views', path.join(__dirname, 'app_server', 'views'));
app.set('view engine', 'hbs');

hbs.registerPartials(
  path.join(__dirname, 'app_server', 'views', 'layouts')
);

app.use(logger('dev'));
app.use(express.json());
app.use(express.urlencoded({ extended: false }));
app.use(cookieParser());
app.use(express.static(path.join(__dirname, 'public')));

app.use('/', indexRouter);


// catch 404 and forward to error handler
app.use(function(req, res, next) {
  next(createError(404));
});

// error handler
app.use(errorHandler);

module.exports = app;

---

## Enhancement Two: Algorithms and Data Structures

This enhancement focuses on improving how data is retrieved and processed in the Travlr Getaways application.

The original implementation returned all trip data at once without considering performance or scalability. To improve this, I added pagination and sorting to the trip retrieval route. This allows the application to return smaller sets of data and organize results more efficiently.

This enhancement demonstrates algorithmic thinking by improving how data is handled and reducing unnecessary processing. It also makes the application more scalable and better suited for handling larger datasets.

**Key Improvements:**
- Added pagination using limit and offset
- Implemented sorting for consistent data ordering
- Improved efficiency of data retrieval

---

## Enhancement Three: Databases

This enhancement focuses on improving the database structure and data integrity in the Travlr Getaways application.

The original implementation used basic schema definitions without much validation. To improve this, I enhanced the Trip model by adding validation rules, trimming input values, default values, and timestamps. These changes ensure that data stored in the database is more consistent and reliable.

This enhancement demonstrates database design skills by improving how data is validated and managed. It also strengthens the overall reliability of the application by preventing incorrect or poorly formatted data from being stored.

**Key Improvements:**
- Added schema validation rules
- Improved data consistency
- Added timestamps for tracking changes
- Strengthened data integrity
