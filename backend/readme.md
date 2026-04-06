The backend in Bhaav Book is a small Express + MongoDB API that sits behind the frontend and handles user profiles, journal entries, AI insights, and user search.

**How it’s structured**
- Entry point: [index.js](F:\Jagriti\Bhaav-Book\backend\index.js)
  It creates the Express app, loads env vars, connects to MongoDB, configures CORS, mounts routes, and exports the app for Vercel deployment instead of starting a local server by default.
- Routes: [routes](F:\Jagriti\Bhaav-Book\backend\routes)
  These define the API endpoints and forward requests to controllers or inline handlers.
- Controllers: [controllers](F:\Jagriti\Bhaav-Book\backend\controllers)
  These contain the main request logic for auth, journals, and insights.
- Models: [models](F:\Jagriti\Bhaav-Book\backend\models)
  Mongoose schemas for `User`, `Journal`, and `Insight`.
- Middlewares: [middlewares](F:\Jagriti\Bhaav-Book\backend\middlewares)
  Firebase auth verification and `multer` upload handling.
- Utils/services: [utils](F:\Jagriti\Bhaav-Book\backend\utils), [services](F:\Jagriti\Bhaav-Book\backend\services)
  DB connection, Firebase Admin setup, Cloudinary upload, DataURI conversion, Gemini AI integration, and the combined insight-generation workflow.

**Request flow**
1. A request hits a route like `/api/journal` or `/api/auth`.
2. Protected routes use [isAuthenticated.js](F:\Jagriti\Bhaav-Book\backend\middlewares\isAuthenticated.js), which verifies the Firebase ID token and loads the matching MongoDB user into `req.user`.
3. Controllers read/write MongoDB through the Mongoose models.
4. For uploads, [multer.js](F:\Jagriti\Bhaav-Book\backend\middlewares\multer.js) stores files in memory, then controllers upload them to Cloudinary.
5. For insights, the controller calls [combinedInsight.service.js](F:\Jagriti\Bhaav-Book\backend\services\combinedInsight.service.js), which collects recent journals, builds one combined prompt, sends it to Gemini through [geminiAi.js](F:\Jagriti\Bhaav-Book\backend\utils\geminiAi.js), then saves the returned summary into the `Insight` collection.

**Main features**
- Auth/profile
  [auth.route.js](F:\Jagriti\Bhaav-Book\backend\routes\auth.route.js) and [firebase.route.js](F:\Jagriti\Bhaav-Book\backend\routes\firebase.route.js) manage Firebase-based signup/login, profile fetch, profile update, logout, and public profile lookup.
- Journals
  [journal.route.js](F:\Jagriti\Bhaav-Book\backend\routes\journal.route.js) and [journal.controller.js](F:\Jagriti\Bhaav-Book\backend\controllers\journal.controller.js) support create, read, update, and delete for entries. Creating an entry also updates the user’s `entryCount`, `streak`, and `lastLoggedDate`.
- Insights
  [insight.route.js](F:\Jagriti\Bhaav-Book\backend\routes\insight.route.js) and [insight.controller.js](F:\Jagriti\Bhaav-Book\backend\controllers\insight.controller.js) generate and store AI summaries over a time window or recent entries.
- Search
  [search.route.js](F:\Jagriti\Bhaav-Book\backend\routes\search.route.js) searches users by name or exact email and returns lightweight profile data.

**Data model overview**
- [user.model.js](F:\Jagriti\Bhaav-Book\backend\models\user.model.js)
  Stores identity fields plus a nested `profile` object for avatar, bio, streaks, preferences, timezone, and insight frequency.
- [journal.model.js](F:\Jagriti\Bhaav-Book\backend\models\journal.model.js)
  Stores one journal entry with title, content, emotions, mood, tags, location, weather, photo, and optional insight text.
- [insight.model.js](F:\Jagriti\Bhaav-Book\backend\models\insight.model.js)
  Stores generated summaries, sentiment, suggestions, mistakes, stuck-state analysis, and linked journals.

**Important environment dependencies**
- MongoDB via `MONGO_URI`
- Firebase Admin via `GOOGLE_SERVICE_ACCOUNT` as base64 JSON
- Cloudinary credentials
- Gemini API key
- JWT secret for one Firebase login path

A few notable quirks I noticed:
- The backend is clearly built primarily for Vercel/serverless use, since `app.listen(...)` is commented out in [index.js](F:\Jagriti\Bhaav-Book\backend\index.js).
- There’s some duplicated auth logic between the auth controller and Firebase routes.
- [package.json](F:\Jagriti\Bhaav-Book\backend\package.json) includes both Firebase and JWT flows, but most protected APIs actually trust Firebase tokens.
- Some debug/experimental code is still present, especially in Firebase auth and Gemini integration.

If you want, I can do the next step and give you either:
1. a route-by-route API map,
2. a backend code review with issues/risk spots, or
3. a simple diagram of how frontend talks to this backend.