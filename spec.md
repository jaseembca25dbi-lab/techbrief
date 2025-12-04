# TechBrief - Full Specification

## Overview

TechBrief is a fullstack news aggregator that fetches RSS feeds from multiple sources, clusters similar stories, generates AI summaries, and presents them in a colorful mood-based interface.

## Architecture

### Frontend (React + Vite + Tailwind)

**Pages:**
- `/` - Auth page (login/register)
- `/feed` - Main feed with search, filters, bookmarking
- `/story/:id` - Full story detail with all sources
- `/bookmarks` - User's saved stories
- `/analytics` - Simple charts showing top tags and stories

**Components:**
- `StoryCard` - Displays story with title, summary, tags, bookmark button, mood theme
- `TagChip` - Clickable tag badge
- `FilterPanel` - Tag filter checkboxes
- `SearchBar` - Search input with debounce
- `Loader` - Loading skeleton
- `ErrorState` - Error message display
- `ThemeToggle` - Dark/light mode switcher
- `Navbar` - Navigation with auth state

**Styling:**
- Tailwind CSS with custom color variables
- Mood themes: blue/rainy, yellow/sunny, purple/mysterious, green/fresh
- Gradient backgrounds per story tag
- Dark mode support

**State Management:**
- React Context for auth state
- Local state for feed data
- JWT stored in httpOnly cookie (managed by backend)

### Backend (Node.js + Express + SQLite)

**Database Schema:**

```sql
-- users table
CREATE TABLE users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- rss_feeds table
CREATE TABLE rss_feeds (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  url TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL,
  category TEXT,
  last_fetched_at DATETIME,
  active BOOLEAN DEFAULT 1
);

-- raw_items table (all fetched RSS items)
CREATE TABLE raw_items (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  feed_id INTEGER,
  title TEXT NOT NULL,
  link TEXT UNIQUE NOT NULL,
  content TEXT,
  published_at DATETIME,
  fetched_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  cluster_id INTEGER,
  FOREIGN KEY (feed_id) REFERENCES rss_feeds(id)
);

-- stories table (canonical clustered stories)
CREATE TABLE stories (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  cluster_id INTEGER UNIQUE NOT NULL,
  canonical_title TEXT NOT NULL,
  summary TEXT,
  tags TEXT, -- JSON array
  source_count INTEGER DEFAULT 1,
  first_seen_at DATETIME,
  last_updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- bookmarks table
CREATE TABLE bookmarks (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  user_id INTEGER NOT NULL,
  story_id INTEGER NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (user_id) REFERENCES users(id),
  FOREIGN KEY (story_id) REFERENCES stories(id),
  UNIQUE(user_id, story_id)
);

-- analytics table (simple counters)
CREATE TABLE analytics (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  event_type TEXT NOT NULL, -- 'view', 'bookmark', 'search'
  entity_id INTEGER,
  metadata TEXT, -- JSON
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

**API Endpoints:**

1. **POST /api/auth/register**
   - Body: `{email, password}`
   - Returns: JWT cookie, user object
   - Validates email format, password length (min 6 chars)
   - Hashes password with bcrypt

2. **POST /api/auth/login**
   - Body: `{email, password}`
   - Returns: JWT cookie, user object
   - Verifies password hash

3. **POST /api/auth/logout**
   - Clears JWT cookie

4. **GET /api/feed**
   - Query params: `tags` (comma-separated), `q` (search), `page` (default 1)
   - Returns: `{stories: [], total, page, pageSize}`
   - Stories include: id, title, summary, tags, sourceCount, publishedAt

5. **GET /api/story/:id**
   - Returns: Full story with all raw_items (sources)
   - Includes: canonical story + array of source articles

6. **POST /api/bookmark**
   - Auth required
   - Body: `{storyId}`
   - Toggles bookmark (save/unsave)

7. **GET /api/bookmarks**
   - Auth required
   - Returns: Array of bookmarked stories

8. **GET /api/analytics**
   - Returns: `{topTags: [{tag, count}], topStories: [{id, title, views}]}`

9. **POST /api/ingest** (internal)
   - Triggers RSS feed fetch and processing
   - Fetches all active feeds
   - Normalizes items
   - Clusters similar stories
   - Generates summaries
   - Updates database

10. **POST /api/mood-sample** (placeholder)
    - Stub for future mood-based recommendations

**Services:**

1. **rssService.js**
   - `fetchAllFeeds()` - Fetches all active RSS feeds
   - `normalizeItem(rawItem)` - Converts RSS item to standard format
   - `sanitizeContent(html)` - Strips dangerous HTML (XSS protection)

2. **summaryService.js**
   - `summarize(text)` - Generates summary (stub with OpenAI example)
   - Temperature: 0 for consistency
   - Max tokens: 150

3. **clusterService.js**
   - `clusterItems(items)` - Groups similar stories
   - Uses fuzzy title matching + token overlap
   - Returns cluster assignments

4. **storyStore.js**
   - `saveStory(story)` - Inserts/updates canonical story
   - `getStories(filters)` - Queries with pagination
   - `getStoryById(id)` - Gets full story with sources

5. **authService.js**
   - `hashPassword(password)` - Bcrypt hash
   - `verifyPassword(password, hash)` - Bcrypt compare
   - `generateToken(user)` - JWT signing
   - `verifyToken(token)` - JWT verification

6. **analyticsService.js**
   - `trackEvent(type, entityId, metadata)` - Logs event
   - `getTopTags()` - Aggregates tag counts
   - `getTopStories()` - Aggregates story views

**Middleware:**

1. **authMiddleware.js**
   - Verifies JWT from cookie
   - Attaches `req.user` if valid
   - Returns 401 if invalid/missing

2. **errorHandler.js**
   - Catches all errors
   - Returns consistent JSON error responses

## Data Flow

1. **Ingestion:**
   - Cron/manual trigger → `POST /api/ingest`
   - Fetch RSS feeds → normalize items → cluster → summarize → store

2. **Feed Display:**
   - User visits `/feed` → `GET /api/feed?tags=tech&page=1`
   - Backend queries stories table with filters
   - Returns paginated results

3. **Bookmarking:**
   - User clicks bookmark → `POST /api/bookmark {storyId}`
   - Backend inserts/deletes bookmark record
   - Analytics event logged

4. **Analytics:**
   - User visits `/analytics` → `GET /api/analytics`
   - Backend aggregates from analytics table
   - Returns top tags and stories

## Security

- Passwords hashed with bcrypt (10 rounds)
- JWT stored in httpOnly cookie (not accessible to JS)
- CORS configured for frontend origin
- RSS content sanitized to prevent XSS
- SQL injection prevented by parameterized queries (Knex)
- Rate limiting recommended for production

## Seed Data

The seed script (`backend/seeds/001_initial_data.js`) inserts:

1. **Sample RSS Feeds:**
   - Hacker News RSS
   - Reddit /r/programming
   - GitHub Trending (daily)
   - TechCrunch
   - Ars Technica
   - The Verge

2. **Demo User:**
   - Email: demo@techbrief.com
   - Password: demo123

3. **Initial Ingest:**
   - Fetches feeds and creates ~20-30 sample stories

## Theme System

**Mood Colors (Tailwind config):**

```js
colors: {
  sunny: { light: '#FEF3C7', DEFAULT: '#FCD34D', dark: '#F59E0B' },
  rainy: { light: '#DBEAFE', DEFAULT: '#60A5FA', dark: '#2563EB' },
  mysterious: { light: '#E9D5FF', DEFAULT: '#A78BFA', dark: '#7C3AED' },
  fresh: { light: '#D1FAE5', DEFAULT: '#34D399', dark: '#059669' }
}
```

**Tag → Mood Mapping:**
- tech, programming, code → rainy (blue)
- startup, business, money → sunny (yellow)
- security, privacy, crypto → mysterious (purple)
- science, health, environment → fresh (green)

## Future Enhancements

- Real-time updates with WebSockets
- Email digest subscriptions (SMTP integration)
- Advanced clustering with embeddings
- Sentiment analysis for mood detection
- Mobile app (React Native)
- Social features (comments, sharing)
- Personalized recommendations
- Multi-language support

## Development Workflow

1. Make changes to code
2. Backend auto-reloads (nodemon)
3. Frontend hot-reloads (Vite HMR)
4. Test endpoints with included examples
5. Check database with SQLite viewer

## Deployment Checklist

- [ ] Change JWT_SECRET
- [ ] Set NODE_ENV=production
- [ ] Enable HTTPS
- [ ] Add rate limiting
- [ ] Configure CORS for production domain
- [ ] Set up database backups
- [ ] Add monitoring (error tracking)
- [ ] Optimize bundle size
- [ ] Add CDN for static assets
- [ ] Set up CI/CD pipeline
