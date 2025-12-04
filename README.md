# TechBrief

A fullstack news aggregator that fetches RSS feeds, clusters similar stories, generates summaries, and provides a colorful mood-based reading experience.

## Features

- 📰 RSS feed aggregation with intelligent clustering
- 🎨 Colorful mood-based themes (sunny, rainy, etc.)
- 🔖 Bookmark your favorite stories
- 📊 Analytics dashboard with top tags and stories
- 🔐 Secure authentication with JWT
- 🌓 Dark/light theme toggle
- 🔍 Search and filter by tags

## Tech Stack

**Frontend:** React + Vite + Tailwind CSS  
**Backend:** Node.js + Express + SQLite  
**Auth:** JWT with httpOnly cookies  
**Database:** SQLite with Knex.js

## Quick Start

### Prerequisites

- Node.js 18+ and npm
- Git

### Installation

```bash
# Clone the repository
git clone <your-repo-url>
cd TechBrief

# Install backend dependencies
cd backend
npm install

# Install frontend dependencies
cd ../frontend
npm install
```

### Environment Setup

Create `backend/.env` file:

```env
# Database
DATABASE_URL=sqlite:./data/dev.sqlite

# Auth (change in production!)
JWT_SECRET=change_me_to_a_secure_random_string

# Optional: OpenAI for real summaries (stub by default)
# OPENAI_KEY=sk-...

# Optional: SMTP for email digests (disabled by default)
# SMTP_URL=smtp://user:pass@smtp.example.com:587

# RSS Feeds (seed script provides defaults)
# Add custom feeds in the database after seeding
```

### Database Setup

```bash
cd backend
npm run db:migrate
npm run db:seed
```

This creates tables and seeds 6 sample RSS feeds (Hacker News, Reddit, GitHub Trending, etc.) with initial stories.

### Running the Application

**Option 1: Run both servers together (recommended)**

```bash
# From project root
npm run dev
```

**Option 2: Run separately**

```bash
# Terminal 1 - Backend (runs on http://localhost:3000)
cd backend
npm run dev
88700
# Terminal 2 - Frontend (runs on http://localhost:5173)
cd frontend
npm run dev
```

Visit http://localhost:5173 to see the app!

## Default Test Account

After seeding, you can register a new account or use:
- Email: `demo@techbrief.com`
- Password: `demo123`

(Created by seed script)

## Project Structure

```
TechBrief/
├── backend/
│   ├── src/
│   │   ├── routes/          # API endpoints
│   │   ├── services/        # Business logic
│   │   ├── middleware/      # Auth, validation
│   │   └── index.js         # Express app
│   ├── migrations/          # Database migrations
│   ├── seeds/               # Sample data
│   └── data/                # SQLite database file
├── frontend/
│   ├── src/
│   │   ├── pages/           # Route pages
│   │   ├── components/      # Reusable components
│   │   ├── services/        # API calls
│   │   └── App.jsx          # Main app
│   └── public/              # Static assets
└── .kiro/
    └── spec.md              # Full specification
```

## API Endpoints

### Auth
- `POST /api/auth/register` - Create account
- `POST /api/auth/login` - Login
- `POST /api/auth/logout` - Logout

### Feed
- `GET /api/feed?tags=tech&q=search&page=1` - Get paginated stories
- `GET /api/story/:id` - Get full story details

### Bookmarks
- `POST /api/bookmark` - Save/unsave bookmark (requires auth)
- `GET /api/bookmarks` - Get user's bookmarks (requires auth)

### Analytics
- `GET /api/analytics` - Get top tags and stories

### Internal
- `POST /api/ingest` - Trigger RSS feed ingestion (internal)

## Development Notes

### Extending with Real AI

The project includes stubs for AI integration. To add real summarization:

1. Add your OpenAI API key to `.env`
2. Uncomment the OpenAI code in `backend/src/services/summaryService.js`
3. Install: `npm install openai`

### Adding More RSS Feeds

Insert into the `rss_feeds` table:

```sql
INSERT INTO rss_feeds (url, name, category) 
VALUES ('https://example.com/feed.xml', 'Example Blog', 'tech');
```

Then run: `npm run ingest` (or call `POST /api/ingest`)

### Security Hardening for Production

- Change `JWT_SECRET` to a strong random value
- Enable HTTPS
- Add rate limiting
- Strengthen XSS sanitization (see comments in `rssService.js`)
- Use environment-specific configs
- Add CORS restrictions

## Hackathon Tips

- The seed data gives you instant content to work with
- Stub services are marked with `// TODO: Replace with real implementation`
- Theme colors are in `tailwind.config.js` - customize away!
- Lottie animation placeholders are in story cards - drop in real animations

## License

MIT
