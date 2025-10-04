# SmartHemo - AI-Powered Test Strip Analysis

## Overview

SmartHemo is an AI-powered application for analyzing medical test strips (specifically glucose test strips) using computer vision and machine learning. The application captures images through a device camera, performs color analysis using TensorFlow.js models, and provides glucose level predictions with confidence scores. It includes calibration features to improve accuracy under different lighting conditions and environmental factors.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Frontend Architecture

**Framework**: React with TypeScript using Vite as the build tool

**UI Component Library**: Shadcn UI built on Radix UI primitives with Tailwind CSS for styling

**State Management**: 
- TanStack Query (React Query) for server state and data fetching
- React hooks for local component state
- No global state management library

**Routing**: Wouter for lightweight client-side routing

**Key Design Patterns**:
- Component composition with custom hooks for reusable logic
- Separation of concerns with dedicated hooks for camera operations (`use-camera.ts`) and color analysis (`use-color-analysis.ts`)
- Data fetching abstracted through a query client with custom API request utilities

### Backend Architecture

**Framework**: Express.js with TypeScript running on Node.js

**API Design**: RESTful API with the following main endpoints:
- `/api/calibration-samples` - Manage calibration data samples
- `/api/test-results` - Store and retrieve test results
- `/api/environment/current` - Environmental readings (temperature, humidity, light)

**Storage Layer**: 
- In-memory storage implementation (`MemStorage` class) for development
- Designed with an `IStorage` interface to allow easy swapping to database persistence
- Drizzle ORM configured for PostgreSQL (ready for database integration)

**Server-Side Rendering**: Vite middleware integration for development with HMR support

### Data Storage Solutions

**Current Implementation**: In-memory storage using Map data structures

**Planned Implementation**: PostgreSQL database via Drizzle ORM

**Database Schema** (defined in `shared/schema.ts`):
- `calibration_samples`: Stores reference color samples with true colors, detected colors, lighting conditions, and captured images
- `test_results`: Stores test strip analysis results including detected colors, glucose ranges, confidence scores, and images
- `environment_readings`: Environmental sensor data (temperature, humidity, light intensity)

**Schema Design Rationale**:
- JSONB fields for flexible storage of detected color arrays
- Decimal types for precise numeric measurements
- Timestamps for temporal analysis
- Base64 encoded images stored as text for simplicity (trade-off: storage size vs. implementation complexity)

### Machine Learning Integration

**Framework**: TensorFlow.js for browser-based and Node.js-based ML

**Models** (foundational structure in place):
1. **Color Intensity Model**: Predicts glucose levels from color intensities
2. **Strip Detection Model**: YOLO-style object detection to locate test strips in images
3. **Light Condition Classifier**: Classifies ambient lighting conditions

**ML Architecture Decisions**:
- Client-side inference for real-time feedback and reduced server load
- Model files would be served statically and loaded on demand
- Fallback to mock predictions during development (actual models require training)

**Rationale**: Browser-based ML reduces latency, works offline, and maintains user privacy by keeping image data on device

### Image Processing Pipeline

**Color Analysis Workflow**:
1. Camera capture via MediaStream API
2. Canvas-based image processing to extract ImageData
3. Color extraction from specific regions of interest
4. Calibration color matching using stored reference samples
5. ML model inference for glucose prediction
6. Confidence scoring and result validation

**Camera Configuration**:
- Support for manual/auto focus modes
- Exposure control
- Multiple lighting condition presets (LED, Daylight, Fluorescent, Incandescent)
- Environment-facing camera preferred for strip capture

### Calibration System

**Purpose**: Improve color detection accuracy across different devices and lighting conditions

**Mechanism**:
- Users capture reference images of known color standards
- System stores true colors alongside detected colors
- Color matching algorithm finds closest calibration sample
- Accuracy metrics calculated based on calibration dataset

**Dataset**: Pre-loaded JSON calibration dataset in `attached_assets/` with sample images

## External Dependencies

### Third-Party Libraries

**UI Components**:
- Radix UI primitives for accessible component foundations
- Lucide React for iconography
- date-fns for date formatting

**Data & State**:
- TanStack Query for async state management
- React Hook Form with Zod resolvers for form validation
- Drizzle ORM for database operations

**Machine Learning**:
- TensorFlow.js core and Node.js bindings
- Note: Models are placeholder structures requiring actual training data

**Development Tools**:
- Vite with React plugin for fast builds
- Replit-specific plugins (cartographer, dev banner, runtime error overlay) for cloud development environment
- TSX for TypeScript execution in development

### Database Services

**Configured For**: Neon Database (serverless PostgreSQL)
- Connection via `@neondatabase/serverless` package
- Connection pooling with `connect-pg-simple` for session management
- Environment variable `DATABASE_URL` required for connection

**Migration Strategy**: Drizzle Kit for schema migrations in `./migrations` directory

### API Integrations

**Current State**: No external API dependencies beyond database

**Potential Future Integrations**:
- Cloud storage for images (if moving away from base64 encoding)
- ML model hosting service
- Analytics/monitoring services

### Build & Deployment

**Build Process**:
- Frontend: Vite builds React app to `dist/public`
- Backend: esbuild bundles Express server to `dist/index.js`
- Format: ESM modules throughout

**Environment Requirements**:
- Node.js with ESM support
- PostgreSQL database (when moving from in-memory storage)
- Camera-enabled device for full functionality