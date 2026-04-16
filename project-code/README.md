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

router.get('/trips', async (req, res) => {
  try {
    const limit = parseInt(req.query.limit, 10) || 10;
    const offset = parseInt(req.query.offset, 10) || 0;

    if (limit < 1 || offset < 0) {
      return res.status(400).json({
        message: 'Invalid pagination values.'
      });
    }

    const trips = await Trip.find({})
      .sort({ name: 1 })
      .skip(offset)
      .limit(limit)
      .exec();

    return res.status(200).json(trips);
  } catch (err) {
    return res.status(500).json({
      message: 'Failed to retrieve trips.',
      error: err?.message ?? err
    });
  }
});

const mongoose = require('mongoose');

const tripSchema = new mongoose.Schema(
  {
    name: {
      type: String,
      required: true,
      trim: true,
      minlength: 3
    },
    length: {
      type: String,
      required: true,
      trim: true
    },
    resort: {
      type: String,
      required: true,
      trim: true
    },
    perPerson: {
      type: String,
      required: true,
      trim: true,
      validate: {
        validator: function(value) {
          return /^\$?\d+(\.\d{2})?$/.test(value);
        },
        message: 'perPerson must be a valid price format.'
      }
    },
    image: {
      type: String,
      required: true,
      trim: true,
      default: 'default-trip.jpg'
    },
    description: {
      type: String,
      required: true,
      trim: true,
      minlength: 10
    }
  },
  {
    timestamps: true
  }
);

mongoose.model('Trip', tripSchema);

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

### errorHandler.js

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
