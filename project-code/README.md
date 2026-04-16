# Project Code (Enhanced)

This folder contains selected files from the Travlr Getaways application that demonstrate the enhancements completed during the CS 499 capstone.

---

## Software Design (Milestone 2)

Centralized error handling

**Files:**
- app.js
- errorHandler.js

### app.js

```javascript
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


function errorHandler(err, req, res, next) {
  console.error(err);

  res.status(err.status || 500);

  if (req.originalUrl.startsWith('/api')) {
    return res.json({
      message: err.message || "Internal Server Error",
      status: err.status || 500
    });
  }

  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  res.render('error');
}

module.exports = errorHandler;
