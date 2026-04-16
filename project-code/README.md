# Project Code (Enhanced)

This folder contains selected files from the Travlr Getaways application that demonstrate the enhancements completed during the CS 499 capstone.

## Included Enhancements

- **Software Design (Milestone 2):**
  - Centralized error handling
  - Files: app.js, errorHandler.js

### app.js
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

### errorhandler.js
// Centralized error handler for the application
function errorHandler(err, req, res, next) {
  
  // Log the error to the console for debugging
  console.error(err);

  // Set the HTTP status code
  res.status(err.status || 500);

  // If the request is for an API route, return a JSON response
  if (req.originalUrl.startsWith('/api')) {
    return res.json({
      message: err.message || "Internal Server Error",
      status: err.status || 500
    });
  }

  // Otherwise render the default error page
  res.locals.message = err.message;
  res.locals.error = req.app.get('env') === 'development' ? err : {};

  res.render('error');
}

module.exports = errorHandler;


- **Algorithms & Data Structures (Milestone 3):**
  - Pagination and sorting added to trip retrieval
  - File: routes-trips.js

### routes-trips.js
/**
 * routes/trips.js
 * Module 7: Adds JWT authentication (register/login) and secures admin CRUD endpoints.
 *
 * Expected mount:
 *   app.use('/api', require('./routes/trips'));
 *
 * Endpoints:
 *   POST   /api/register
 *   POST   /api/login
 *
 *   GET    /api/trips              (public)
 *   GET    /api/trips/:tripId      (protected)
 *   POST   /api/trips              (protected)
 *   PUT    /api/trips/:tripId      (protected)
 *   DELETE /api/trips/:tripId      (protected)
 */

const express = require('express');
const router = express.Router();
const mongoose = require('mongoose');
const jwt = require('jsonwebtoken');
const { jwtSecret } = require('../config');

// Ensure User model is registered
require('../models/user');
const User = mongoose.model('User');

// Trip model must already be registered in your app
const Trip = mongoose.model('Trip');

/**
 * CORS middleware for Angular dev server (localhost:4200).
 * Allows Authorization header for JWT and handles OPTIONS preflight.
 */
router.use((req, res, next) => {
  res.header('Access-Control-Allow-Origin', '*');
  res.header('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE,OPTIONS');
  res.header('Access-Control-Allow-Headers', 'Content-Type, Authorization');

  if (req.method === 'OPTIONS') {
    return res.sendStatus(204);
  }
  next();
});

/**
 * JWT auth middleware
 */
function requireAuth(req, res, next) {
  const header = req.headers.authorization || '';
  const token = header.startsWith('Bearer ') ? header.slice(7) : null;

  if (!token) {
    return res.status(401).json({ message: 'Unauthorized: missing token' });
  }

  jwt.verify(token, jwtSecret, (err, payload) => {
    if (err) {
      return res.status(401).json({ message: 'Unauthorized: invalid token' });
    }
    req.user = payload;
    next();
  });
}

/**
 * AUTH: Register
 * POST /api/register
 * Body: { name, email, password }
 */
router.post('/register', async (req, res) => {
  try {
    const name = (req.body.name || '').trim();
    const email = (req.body.email || '').trim().toLowerCase();
    const password = req.body.password || '';

    if (!email || !password) {
      return res.status(400).json({ message: 'Email and password are required.' });
    }

    const existing = await User.findOne({ email }).exec();
    if (existing) {
      return res.status(409).json({ message: 'User already exists.' });
    }

    const user = new User({
      name: name || email,
      email,
      hash: 'temp',
      salt: 'temp'
    });

    user.setPassword(password);
    await user.save();

    return res.status(201).json({ token: user.generateJwt() });
  } catch (err) {
    return res.status(500).json({
      message: 'Registration failed.',
      error: err?.message ?? err
    });
  }
});

/**
 * AUTH: Login
 * POST /api/login
 * Body: { email, password }
 */
router.post('/login', async (req, res) => {
  try {
    const email = (req.body.email || '').trim().toLowerCase();
    const password = req.body.password || '';

    if (!email || !password) {
      return res.status(400).json({ message: 'Email and password are required.' });
    }

    const user = await User.findOne({ email }).exec();
    if (!user || !user.validPassword(password)) {
      return res.status(401).json({ message: 'Invalid email or password.' });
    }

    return res.status(200).json({ token: user.generateJwt() });
  } catch (err) {
    return res.status(500).json({
      message: 'Login failed.',
      error: err?.message ?? err
    });
  }
});

/**
 * TRIPS: GET all (public)
 * GET /api/trips
 * Supports pagination and alphabetical sorting
 */
router.get('/trips', async (req, res) => {
  try {
    // Parse pagination values from query string
    const limit = parseInt(req.query.limit, 10) || 10;
    const offset = parseInt(req.query.offset, 10) || 0;

    // Validate pagination values
    if (limit < 1 || offset < 0) {
      return res.status(400).json({
        message: 'Invalid pagination values.'
      });
    }

    // Retrieve trips using pagination and sorting by name
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

/**
 * TRIPS: GET one (protected)
 * GET /api/trips/:tripId
 */
router.get('/trips/:tripId', requireAuth, async (req, res) => {
  try {
    const { tripId } = req.params;

    if (!mongoose.Types.ObjectId.isValid(tripId)) {
      return res.status(400).json({ message: 'Invalid tripId format.' });
    }

    const trip = await Trip.findById(tripId).exec();
    if (!trip) {
      return res.status(404).json({ message: 'Trip not found.' });
    }

    return res.status(200).json(trip);
  } catch (err) {
    return res.status(500).json({
      message: 'Failed to retrieve trip.',
      error: err?.message ?? err
    });
  }
});

/**
 * TRIPS: POST create (protected)
 * POST /api/trips
 */
router.post('/trips', requireAuth, async (req, res) => {
  try {
    const perPerson = req.body.perPerson ?? req.body.price;

    const newTrip = {
      code: req.body.code,
      name: req.body.name,
      length: req.body.length,
      start: req.body.start,
      resort: req.body.resort,
      perPerson: perPerson,
      price: req.body.price ?? perPerson,
      image: req.body.image,
      description: req.body.description
    };

    const created = await Trip.create(newTrip);
    return res.status(201).json(created);
  } catch (err) {
    if (err?.name === 'ValidationError') {
      return res.status(400).json(err);
    }
    return res.status(500).json({
      message: 'Failed to create trip.',
      error: err?.message ?? err
    });
  }
});

/**
 * TRIPS: PUT update (protected)
 * PUT /api/trips/:tripId
 */
router.put('/trips/:tripId', requireAuth, async (req, res) => {
  try {
    const { tripId } = req.params;

    if (!mongoose.Types.ObjectId.isValid(tripId)) {
      return res.status(400).json({ message: 'Invalid tripId format.' });
    }

    const perPerson = req.body.perPerson ?? req.body.price;

    const update = {
      code: req.body.code,
      name: req.body.name,
      length: req.body.length,
      start: req.body.start,
      resort: req.body.resort,
      image: req.body.image,
      description: req.body.description
    };

    if (perPerson !== undefined) {
      update.perPerson = perPerson;
      update.price = req.body.price ?? perPerson;
    }

    Object.keys(update).forEach((k) => update[k] === undefined && delete update[k]);

    const updated = await Trip.findByIdAndUpdate(tripId, update, {
      new: true,
      runValidators: true
    }).exec();

    if (!updated) {
      return res.status(404).json({ message: 'Trip not found.' });
    }

    return res.status(200).json(updated);
  } catch (err) {
    if (err?.name === 'ValidationError') {
      return res.status(400).json(err);
    }

    return res.status(500).json({
      message: 'Failed to update trip.',
      error: err?.message ?? err
    });
  }
});

/**
 * TRIPS: DELETE (protected)
 * DELETE /api/trips/:tripId
 */
router.delete('/trips/:tripId', requireAuth, async (req, res) => {
  try {
    const { tripId } = req.params;

    if (!mongoose.Types.ObjectId.isValid(tripId)) {
      return res.status(400).json({ message: 'Invalid tripId format.' });
    }

    const deleted = await Trip.findByIdAndDelete(tripId).exec();
    if (!deleted) {
      return res.status(404).json({ message: 'Trip not found.' });
    }

    return res.sendStatus(204);
  } catch (err) {
    return res.status(500).json({
      message: 'Failed to delete trip.',
      error: err?.message ?? err
    });
  }
});

module.exports = router;

- **Databases (Milestone 4):**
  - Schema validation and data integrity improvements
  - File: models-trips.js

### models-trips.js
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


