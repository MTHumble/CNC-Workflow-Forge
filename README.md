# CNC Workflow Forge

A dual-mode manufacturing operations platform combining training and production systems that share the same codebase. Designed for Applied Stoicism Labs to teach operators real business problems (cash flow, scheduling, quality) before they experience them with real money.

## Quick Start

### Prerequisites
- Node.js 18+ 
- Docker + Docker Compose (for local Supabase)
- Git

### Development Setup

```bash
# Install dependencies
npm install

# Set up environment variables
cp packages/backend/.env.example packages/backend/.env
cp packages/frontend/.env.example packages/frontend/.env

# Start local Supabase (requires Docker)
docker-compose up -d

# Run migrations
npm run migrate

# Start development servers
npm run dev
```

This will start:
- **Frontend**: http://localhost:3001
- **Backend API**: http://localhost:3000
- **Supabase Studio**: http://localhost:54323

## Project Structure

```
packages/
├── backend/       # Fastify REST API
│   ├── src/
│   │   ├── config.ts          # Configuration
│   │   ├── db.ts              # Database client
│   │   ├── services/          # Business logic
│   │   ├── routes/            # API endpoints
│   │   └── integrations/      # Machine connectors (MQTT)
│   └── migrations/            # SQL migration files
├── frontend/      # React + Tailwind UI
│   ├── src/
│   │   ├── components/        # React components
│   │   ├── store.ts           # Zustand state management
│   │   ├── api.ts             # API client
│   │   └── index.css          # Tailwind + custom styles
│   └── index.html
└── shared/        # Shared TypeScript types
    └── src/
        └── types.ts           # Domain models
```

## Core Systems

### 1. Job Lifecycle State Machine
Jobs progress through these states:
```
RFQ_RECEIVED → QUOTED → QUOTE_ACCEPTED → SCHEDULED → 
IN_PRODUCTION → QUALITY_CHECK → COMPLETE → INVOICED → PAID
```

Each transition is enforced in the `JobService` with business logic in the backend.

### 2. Dual-Mode Operation
- **Training Mode**: Simulated environment with no real consequences. Perfect for learning.
- **Production Mode**: Real data, real money. Separate ledger from training.

Mode is set at the session level to prevent data mixing.

### 3. MQTT Machine Integration (Phase 1)
Bambu Lab 3D printers are integrated via MQTT:
- Real-time status updates
- Job progress tracking
- Automatic job completion detection

See `packages/backend/src/integrations/bambuMqtt.ts` for implementation.

### 4. Command Center Dashboard
The main UI shows:
- Live job status by stage
- Machine fleet status and utilization
- Daily KPIs (OTD%, margin, scrap%)
- Cash runway alert
- Quick action buttons for job progression

## Database Schema

Key tables (see `packages/backend/migrations/001_core_tables.sql`):
- `customers` - Customer profiles with behavior scoring
- `rfqs` - Request for Quote
- `quotes` - Price proposals
- `jobs` - Production jobs from accepted quotes
- `machines` - Equipment and status
- `work_orders` - Job work orders
- `inspection_records` - QA results
- `ledger_entries` - Financial transactions (training/production separated)
- `crisis_events` - Training scenarios
- `session_modes` - Training vs Production tracking

## Configuration

### Backend (.env)
```
PORT=3000
SUPABASE_URL=http://localhost:54321
SUPABASE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
MQTT_BROKER_URL=mqtt://localhost:1883
MQTT_USERNAME=admin
MQTT_PASSWORD=password
```

### Frontend (.env)
```
VITE_API_URL=http://localhost:3000
VITE_WS_URL=ws://localhost:3000
```

## API Endpoints

### Jobs
- `GET /api/jobs` - Get all active jobs
- `GET /api/jobs/:jobId` - Get job details
- `PUT /api/jobs/:jobId/schedule` - Schedule job
- `PUT /api/jobs/:jobId/start` - Start production
- `PUT /api/jobs/:jobId/progress` - Update progress
- `PUT /api/jobs/:jobId/complete-production` - Mark production done
- `PUT /api/jobs/:jobId/quality-check` - Record inspection
- `PUT /api/jobs/:jobId/invoice` - Create invoice
- `PUT /api/jobs/:jobId/payment` - Record payment

### Machines
- `GET /api/machines` - Get fleet status
- `POST /api/machines` - Register new machine
- `PUT /api/machines/:machineId/status` - Update status

### RFQs & Quotes
- `GET /api/rfqs` - Get all RFQs
- `GET /api/rfqs/:rfqId` - Get RFQ with quotes
- `POST /api/quotes` - Create quote
- `PUT /api/quotes/:quoteId/accept` - Accept quote

## Build & Deploy

```bash
# Build all packages
npm run build

# Backend only
npm run build -w @cnc-forge/backend

# Frontend only  
npm run build -w @cnc-forge/frontend
```

## Testing

```bash
# Run all tests
npm test

# Watch mode
npm test -- --watch
```

## License

Proprietary - Applied Stoicism Labs

## Author

Built for Steve's Machine Shop - Applied Stoicism Labs
