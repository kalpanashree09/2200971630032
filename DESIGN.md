# QuickLink URL Shortener - Design Document

## Overview

QuickLink is a comprehensive URL shortening application built with React and TypeScript. It provides URL shortening capabilities with advanced analytics, custom expiration times, and detailed activity logging, all running purely on the client-side using localStorage for persistence.

## Architecture & File Structure

```
src/
├── components/           # React components
│   ├── Header.tsx       # Navigation header
│   ├── URLShortener.tsx # Main URL shortening form
│   ├── Analytics.tsx    # Analytics dashboard
│   ├── LogsViewer.tsx   # Activity logs viewer
│   └── RedirectHandler.tsx # Handles short URL redirects
├── context/             # React Context providers
│   └── LoggingContext.tsx # Custom logging middleware
├── hooks/               # Custom React hooks
│   └── useLocalStorage.ts # localStorage abstraction
├── types/               # TypeScript type definitions
│   └── index.ts         # All interface definitions
├── utils/               # Utility functions
│   ├── shortcodeGenerator.ts # Short code generation & validation
│   ├── urlStorage.ts    # localStorage URL management
│   └── analytics.ts     # Analytics utilities
├── App.tsx             # Main application with routing
└── main.tsx            # Application entry point
```

## Component Breakdown

### 1. URLShortener Component
- **Purpose**: Main form for creating shortened URLs
- **Features**:
  - URL validation
  - Custom short code input (optional)
  - Expiration time selection
  - Real-time feedback and error handling
  - Copy-to-clipboard functionality
- **State Management**: Local component state for form data

### 2. Analytics Component
- **Purpose**: Comprehensive analytics dashboard
- **Features**:
  - Summary cards (total clicks, active URLs, expired URLs)
  - URL performance list with click counts
  - Detailed URL analytics with click history
  - Geographic and referrer information
- **Data Sources**: URLStorage and localStorage

### 3. RedirectHandler Component
- **Purpose**: Handles shortened URL redirects
- **Features**:
  - Short code validation
  - Analytics tracking for each click
  - User-friendly redirect experience with countdown
  - Error handling for invalid/expired URLs
- **Flow**: Validates → Tracks → Redirects

### 4. LogsViewer Component
- **Purpose**: System activity monitoring
- **Features**:
  - Filterable log levels (info, warning, error)
  - Search functionality
  - Export logs to JSON
  - Real-time activity tracking
- **Data Source**: LoggingContext

### 5. Header Component
- **Purpose**: Navigation and branding
- **Features**:
  - Responsive navigation menu
  - Active route highlighting
  - Brand logo and identity

## Data Model (localStorage)

### 1. Shortened URLs (`shortened_urls`)
```typescript
interface ShortenedURL {
  id: string;              // Unique identifier
  originalUrl: string;     // Original long URL
  shortCode: string;       // Generated/custom short code
  customCode?: string;     // User-provided custom code
  createdAt: number;       // Creation timestamp
  expiresAt: number;       // Expiration timestamp
  isActive: boolean;       // Active status
}
```

### 2. URL Analytics (`url_analytics`)
```typescript
interface URLAnalytics {
  urlId: string;           // References ShortenedURL.id
  clickCount: number;      // Total click count
  clicks: ClickEvent[];    // Array of individual clicks
  lastClicked?: number;    // Last click timestamp
}

interface ClickEvent {
  timestamp: number;       // Click timestamp
  referrer: string;        // Referrer URL
  userAgent: string;       // Browser user agent
  location?: {             // Geographic information
    country?: string;
    city?: string;
    timezone: string;
  };
}
```

### 3. Activity Logs (`app_logs`)
```typescript
interface LogEntry {
  id: string;              // Unique log identifier
  timestamp: number;       // Log timestamp
  action: string;          // Action type (e.g., 'URL_SHORTENED')
  details: Record<string, any>; // Action-specific details
  level: 'info' | 'warning' | 'error'; // Log level
}
```

## Routing Strategy

The application uses React Router for client-side routing:

- **`/`** - Main URL shortening interface
- **`/analytics`** - Analytics dashboard
- **`/logs`** - Activity logs viewer
- **`/:shortCode`** - Dynamic route for URL redirects

### Redirect Handling
The redirect route (`/:shortCode`) is handled by the RedirectHandler component:
1. Extracts the short code from the URL parameters
2. Validates the short code exists and is not expired
3. Creates analytics entry for the click
4. Provides user-friendly redirect experience
5. Redirects to the original URL

## Custom Logging Middleware

The logging system is implemented using React Context:

### LoggingContext
- **Purpose**: Centralized logging system
- **Features**:
  - Automatic log storage to localStorage
  - Log level categorization
  - Automatic log rotation (keeps last 1000 entries)
  - Real-time log updates across components

### Usage Pattern
```typescript
const { addLog } = useLogging();
addLog('URL_SHORTENED', { originalUrl, shortCode }, 'info');
```

## Key Features Implementation

### 1. Short Code Generation
- Uses alphanumeric characters (A-Z, a-z, 0-9)
- Default length: 6 characters
- Collision detection and regeneration
- Custom code validation (3-20 characters, alphanumeric only)

### 2. URL Validation
- Uses native URL constructor for validation
- Supports all standard URL schemes
- Real-time validation feedback

### 3. Expiration Management
- Default expiration: 30 minutes
- User-configurable expiration times
- Automatic cleanup of expired URLs
- Visual indicators for expired URLs

### 4. Analytics Tracking
- Comprehensive click tracking
- Geographic location detection using browser APIs
- Referrer tracking
- User agent collection
- Real-time analytics updates

### 5. Data Persistence
- All data stored in localStorage
- JSON serialization/deserialization
- Error-resistant storage operations
- Automatic data cleanup

## Security Considerations

### 1. Input Validation
- URL format validation
- Short code character restrictions
- Expiration time bounds checking

### 2. XSS Prevention
- All user inputs are properly escaped
- No direct HTML insertion
- React's built-in XSS protection

### 3. Data Privacy
- All data stored locally (no server transmission)
- No sensitive information collection
- Optional geographic tracking

## Performance Optimizations

### 1. Storage Optimization
- Log rotation to prevent storage bloat
- Efficient data structures
- Lazy loading of analytics data

### 2. UI Optimization
- Responsive design with mobile-first approach
- Smooth animations and transitions
- Debounced search functionality

### 3. Memory Management
- Cleanup of expired URLs
- Efficient re-renders using React best practices
- Minimal bundle size with tree shaking

## Browser Compatibility

- **Modern Browsers**: Full feature support
- **localStorage**: Required for persistence
- **Geolocation API**: Optional, graceful degradation
- **Clipboard API**: Fallback to document.execCommand

## Assumptions Made

1. **Client-Side Only**: No backend server required
2. **localStorage Availability**: Assumes localStorage is available and functional
3. **Modern Browser Support**: Assumes ES6+ support and modern JavaScript APIs
4. **Single Device Usage**: Data is not synchronized across devices
5. **Trust Model**: Assumes users won't manipulate localStorage maliciously
6. **Performance**: Assumes reasonable usage (not thousands of URLs)
7. **Network**: Redirect functionality requires internet connectivity
8. **Permissions**: Assumes clipboard and geolocation permissions when requested

## Future Enhancements

1. **Data Export/Import**: Allow users to backup and restore their data
2. **QR Code Generation**: Generate QR codes for shortened URLs
3. **Bulk Operations**: Support for bulk URL shortening
4. **Advanced Analytics**: More detailed geographic and device analytics
5. **Custom Domains**: Support for custom domain configuration
6. **Password Protection**: Optional password protection for URLs
7. **API Integration**: Optional cloud storage integration
8. **PWA Features**: Service worker for offline functionality

## Development & Deployment

### Development
```bash
npm install
npm run dev
```

### Production Build
```bash
npm run build
npm run preview
```

### Testing Strategy
- Component unit tests
- Integration tests for URL shortening flow
- End-to-end tests for redirect functionality
- localStorage persistence testing

The application is designed to be production-ready, maintainable, and easily deployable to any static hosting service.