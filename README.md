# SlackFlow

> A powerful Slack messaging and scheduling application built with React and Node.js

Here's the updated README with the **Architectural Overview** and **Challenges & Learnings** sections added:

# SlackFlow

> A powerful Slack messaging and scheduling application built with React and Node.js
![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361DAFB)
![Express.js](https://img.shields.io/badge/express.js-%23404d59.svg?style=for-the-badge&logo=express&logoColor=%2361DAFB)
![Slack](https://img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white)
![Node.js](https://img.shields.io/badge/node.js-6DA55F?style=for-the-badge&logo=node.js&logoColor=white)







## 📋 Overview

SlackFlow is a modern web application that streamlines Slack messaging workflows. It provides an intuitive interface for composing, scheduling, and managing Slack messages across multiple channels, with powerful OAuth integration and real-time synchronization.

### ✨ Key Features

- 🔐 **Secure Slack OAuth Integration** - Seamless authentication with your Slack workspace
- 📝 **Message Composition** - Rich text editor with character count and validation
- ⏰ **Message Scheduling** - Schedule messages for future delivery with timezone support
- 📊 **Analytics Dashboard** - Track sent messages, scheduled messages, and active channels
- 🚀 **Real-time Updates** - Live synchronization with Slack workspace data
- 🎨 **Modern UI/UX** - Clean, responsive design built with shadcn/ui components
- 📱 **Mobile Responsive** - Works seamlessly across all device sizes
- 🔄 **Message Management** - Cancel, reschedule, and monitor message delivery status

## 🏗️ Architectural Overview

SlackFlow follows a **full-stack TypeScript architecture** with clear separation between client and server responsibilities:

### OAuth & Authentication Flow
```
User → SlackFlow → Slack OAuth → Authorization → Token Exchange → Secure Storage
```

1. **OAuth Initiation**: Client requests authorization URL from server
2. **Slack Authorization**: User redirects to Slack for workspace permission
3. **Callback Handling**: Server receives authorization code with CSRF protection
4. **Token Exchange**: Server exchanges code for Bot User OAuth Token (`xoxb-`)
5. **Secure Storage**: Tokens encrypted and stored with team/user mapping

### Token Management Strategy
- **Bot Tokens**: Stored securely in SQLite with team/user relationships
- **Session Management**: HTTP-only cookies with CSRF state validation
- **Token Validation**: Automatic token health checks every 30 seconds
- **Graceful Expiry**: Handles token expiration with user-friendly re-authentication

### Scheduled Task Architecture
```
React Scheduler → API Validation → Database Queue → Background Service → Slack API
```

- **Message Queuing**: SQLite-based persistent queue for scheduled messages
- **Background Processing**: Node.js `setInterval` checks for due messages every 60 seconds
- **Timezone Handling**: UTC storage with client-side timezone conversion
- **Delivery Tracking**: Status updates (pending → sent → failed) with retry logic
- **Cancellation Support**: Users can cancel messages before delivery

### API Design Pattern
- **RESTful Endpoints**: Clear resource-based URL structure
- **Middleware Chain**: Authentication → Validation → Business Logic → Response
- **Error Handling**: Comprehensive error catching with user-friendly messages
- **Type Safety**: End-to-end TypeScript with Zod schema validation

## 🧩 Challenges & Learnings

### Major Development Hurdles

#### 1. **Slack OAuth Scope Complexity** 🎯
**Challenge**: The biggest hurdle was navigating Slack's extensive documentation to understand the distinction between user tokens, bot tokens, and the specific scopes required for each operation.

**Key Confusions**:
- Difference between `chat:write` vs `chat:write.public` vs deprecated `chat:write:bot`
- When to use Bot Token Scopes vs User Token Scopes
- Understanding Enterprise Grid vs regular workspace limitations
- OAuth v2 vs legacy OAuth flows

**Solution**:
- Systematically tested each scope combination in a development workspace
- Created a scope mapping document for different use cases
- Implemented proper error handling to surface permission issues clearly
- Used Slack's OAuth test tool to validate token permissions

#### 2. **HTTPS Requirement in Development** 🔒
**Challenge**: Slack requires HTTPS URLs for OAuth redirects, but local development runs on HTTP.

**Solution**: 
- Integrated ngrok for local HTTPS tunneling
- Automated ngrok URL updates in development workflow
- Created clear documentation for developers to handle URL changes

#### 3. **Bot Channel Membership Management** 🤖
**Challenge**: Understanding when and how bots need to be invited to channels, even with appropriate scopes.

**Learnings**:
- `chat:write.public` doesn't eliminate the need for channel membership in all cases
- Private channels always require explicit bot invitation
- Workspace admin settings can override scope permissions

#### 4. **Response Stream Consumption Issues** 🔄
**Challenge**: Fetch API response bodies can only be read once, causing "stream already read" errors in error handling.

**Solution**:
- Implemented `Response.clone()` for multiple reads
- Restructured error handling to read once and parse appropriately
- Added proper error boundary patterns

#### 5. **State Management for OAuth Security** 🛡️
**Challenge**: Implementing CSRF protection with state parameter validation while maintaining user experience.

**Solution**:
- HTTP-only cookies for state storage
- Automatic cleanup of expired state tokens
- Clear error messages for state mismatch scenarios

### Key Technical Learnings

#### Slack API Best Practices
```typescript
// Always use Bot User OAuth Tokens (xoxb-) for API calls
const client = new WebClient(process.env.SLACK_BOT_TOKEN);

// Handle rate limiting gracefully
try {
  const result = await client.chat.postMessage({
    channel: channelId,
    text: message
  });
} catch (error) {
  if (error.code === 'rate_limited') {
    // Implement exponential backoff
    await delay(error.retryAfter * 1000);
    // Retry logic here
  }
}
```

#### Token Security Patterns
- Never expose tokens in client-side code
- Use environment variables for sensitive credentials
- Implement token rotation for production environments
- Regular token health checks to catch expired credentials

#### Database Design Insights
- Store team/user relationships for multi-workspace support
- Use UTC timestamps for all scheduling operations
- Implement soft deletes for message history tracking
- Index frequently queried fields (teamId, userId, scheduledFor)

### Development Process Improvements

#### Documentation-First Approach
- Created comprehensive API documentation before implementation
- Maintained a troubleshooting guide throughout development
- Documented all Slack scope requirements and use cases

#### Testing Strategy
- Manual testing with multiple Slack workspaces
- Edge case testing (expired tokens, deleted channels, offline scenarios)
- OAuth flow testing with different user permission levels

#### Error Handling Philosophy
- User-friendly error messages that suggest solutions
- Comprehensive logging for debugging without exposing sensitive data
- Graceful degradation when Slack API is unavailable

These challenges taught valuable lessons about third-party API integration, OAuth security, and building resilient real-time applications. The complexity of Slack's permission system, while initially overwhelming, ultimately led to a deeper understanding of enterprise-grade API design patterns.

## 🛠️ Tech Stack

### Frontend
- **React 18** with TypeScript
- **Wouter** for client-side routing
- **TanStack Query** for state management and data fetching
- **React Hook Form** with Zod validation
- **shadcn/ui** components with Tailwind CSS
- **Vite** for development and building

### Backend
- **Node.js** with Express.js
- **TypeScript** for type safety
- **Slack Web API** for Slack integration
- **SQLite** database with custom storage layer
- **Cookie-based session management**
- **CSRF protection** with state validation

## 📋 Prerequisites

- **Node.js 18.x** or higher
- **npm** or yarn package manager
- **ngrok** installed globally (`npm install -g ngrok`)
- A **Slack workspace** with admin permissions
- **Slack app credentials** (Client ID, Client Secret)

## 🚀 Installation

### 1. Clone the Repository
```bash
git clone https://github.com/beginner1214/SlackFlow.git
cd SlackFlow
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Install and Set Up ngrok
```bash
# Install ngrok globally if not already installed
npm install -g ngrok

# Start ngrok to expose your local server
ngrok http 5000
```

**Important:** Copy the **HTTPS URL** from ngrok output (e.g., `https://abc123.ngrok-free.app`)

### 4. Environment Configuration

Create a `.env` file in the root directory with your ngrok HTTPS URL:

```env
# Slack App Credentials
SLACK_CLIENT_ID=your_slack_client_id
SLACK_CLIENT_SECRET=your_slack_client_secret

# IMPORTANT: Use your ngrok HTTPS URL here
SLACK_REDIRECT_URI=https://abc123.ngrok-free.app/oauth/callback

# Server Configuration
NODE_ENV=development
PORT=5000

# Database (optional - defaults to SQLite)
DATABASE_URL=./data/slackflow.db
```

### 5. Slack App Setup

1. **Create a Slack App**
   - Go to [api.slack.com/apps](https://api.slack.com/apps)
   - Click **"Create New App"** → **"From scratch"**
   - Enter app name (e.g., "SlackFlow") and select your workspace

2. **Configure OAuth & Permissions**
   - Navigate to **"OAuth & Permissions"** in the left sidebar
   - Add **Redirect URLs** (use your ngrok HTTPS URL):
     ```
     https://abc123.ngrok-free.app/oauth/callback
     ```

3. **Add Bot Token Scopes** (Essential):
   - `chat:write` - Send messages as the app
   - `chat:write.public` - Send messages to channels app isn't a member of
   - `channels:read` - View basic information about public channels
   - `channels:join` - Join public channels in a workspace

4. **Install App to Workspace**
   - Scroll up to **"OAuth Tokens for Your Workspace"**
   - Click **"Install to Workspace"**
   - Authorize the app
   - Copy the **Bot User OAuth Token** (starts with `xoxb-`)

5. **Update Environment Variables**
   - Add your Slack credentials to the `.env` file

### 6. Start Development Server

**Two Terminal Setup:**

**Terminal 1 - Keep ngrok running:**
```bash
ngrok http 5000
```

**Terminal 2 - Start the application:**
```bash
npm run dev
```

The application will be available at your **ngrok HTTPS URL** (e.g., `https://abc123.ngrok-free.app`)

## 🔧 Development Workflow

### Every Development Session:
1. **Start ngrok**: `ngrok http 5000`
2. **Copy the HTTPS URL** from ngrok output
3. **Update your `.env` file** with the new ngrok URL (if changed)
4. **Update Slack app redirect URLs** if the ngrok URL changed
5. **Start the dev server**: `npm run dev`

### Important Notes:
- **ngrok URLs change** each time you restart ngrok (unless using a paid static domain)
- **Always use HTTPS URLs** from ngrok, never HTTP
- **Update both** your `.env` file and Slack app settings when ngrok URL changes

## 📖 Usage

### Authentication
1. Open the application at your ngrok HTTPS URL
2. Click **"Connect to Slack"**
3. Authorize the app in your Slack workspace
4. You'll be redirected back to the dashboard

### Composing Messages
1. **Select a channel** from the dropdown
2. **Type your message** (max 4,000 characters)
3. **Choose delivery method:**
   - Send immediately
   - Schedule for later
4. Click **"Send Now"** or **"Schedule Message"**

### Scheduling Messages
1. Check **"Schedule message for later"**
2. Select **date and time**
3. Choose **timezone**
4. Click **"Schedule Message"**

### Managing Scheduled Messages
- View all scheduled messages in the **"Scheduled"** tab
- **Cancel pending messages** before they're sent
- **Monitor delivery status** and history

### Inviting Bot to Channels
If you get a "Bot not in channel" error:
```bash
# In Slack, type in the target channel:
/invite @YourBotName
```

## 🔧 API Endpoints

### Authentication
- `GET /api/slack/auth` - Get Slack OAuth authorization URL
- `POST /api/slack/oauth/callback` - Handle OAuth callback
- `GET /api/slack/status/:teamId/:userId` - Check connection status
- `POST /api/slack/disconnect/:teamId/:userId` - Disconnect Slack account

### Messaging
- `GET /api/slack/channels/:teamId/:userId` - List available channels
- `POST /api/slack/send/:teamId/:userId` - Send immediate message
- `POST /api/messages/schedule/:teamId/:userId` - Schedule message for later

### Message Management
- `GET /api/messages/scheduled/:teamId/:userId` - Get scheduled messages
- `DELETE /api/messages/scheduled/:teamId/:userId/:id` - Cancel scheduled message
- `GET /api/messages/stats/:teamId/:userId` - Get messaging statistics

## 🚨 Troubleshooting

### Common Issues

**"Redirect URI Mismatch" Error**
```bash
# Ensure your .env SLACK_REDIRECT_URI matches your Slack app settings
# Both should use the same ngrok HTTPS URL
SLACK_REDIRECT_URI=https://your-ngrok-url.ngrok-free.app/oauth/callback
```

**"Bot not in channel or insufficient permissions" Error**
```bash
# Invite your bot to the channel in Slack
/invite @YourBotName

# Or add via channel settings:
# Channel → Settings → Integrations → Add apps
```

**ngrok URL Changed**
1. Copy the new ngrok HTTPS URL
2. Update `.env` file with new URL
3. Update Slack app redirect URLs in [api.slack.com/apps](https://api.slack.com/apps)
4. Restart your development server: `npm run dev`

**"404 Page Not Found" Error**
- Make sure you're accessing the **ngrok HTTPS URL**, not `localhost:5000`
- Verify ngrok is properly forwarding to `127.0.0.1:5000`

**SelectItem Empty Value Error**
- Check for any `<SelectItem value="">` components
- Replace with non-empty values like `value="none"`

**"Failed to execute 'text' on 'Response': body stream already read"**
- This indicates a response parsing issue in the error handler
- Restart the development server

**OAuth State Parameter Errors**
- Ensure cookies are enabled in your browser
- Clear browser cookies for the ngrok domain
- Restart the development server

## 🗂️ Project Structure

```
SlackFlow/
├── client/                     # React frontend
│   ├── src/
│   │   ├── components/         # UI components
│   │   │   ├── ui/            # shadcn/ui components
│   │   │   ├── header.tsx     # App header
│   │   │   ├── sidebar.tsx    # Navigation sidebar
│   │   │   ├── compose-section.tsx    # Message composition
│   │   │   └── scheduled-messages.tsx # Scheduled messages view
│   │   ├── hooks/             # Custom React hooks
│   │   │   ├── use-auth.ts    # Authentication hook
│   │   │   └── use-toast.ts   # Toast notifications
│   │   ├── lib/               # Utilities and configurations
│   │   │   ├── queryClient.ts # TanStack Query setup
│   │   │   └── utils.ts       # Utility functions
│   │   ├── pages/             # Route components
│   │   │   ├── dashboard.tsx  # Main dashboard
│   │   │   ├── oauth-callback.tsx # OAuth handler
│   │   │   └── not-found.tsx  # 404 page
│   │   └── main.tsx           # Entry point
│   ├── index.html             # HTML template
│   └── vite.config.ts         # Vite configuration
├── server/                    # Express backend
│   ├── services/              # Business logic
│   │   ├── slack.ts          # Slack API integration
│   │   └── scheduler.ts      # Message scheduling
│   ├── shared/               # Shared types and schemas
│   │   └── schema.ts        # Validation schemas
│   ├── routes.ts            # API routes
│   ├── storage.ts           # Database layer
│   ├── vite.ts              # Vite SSR setup
│   └── index.ts             # Server entry point
├── db/                      # Database files
├── .env                     # Environment variables
├── package.json            # Dependencies
└── README.md              # This file
```

## 🔒 Security Features

- **CSRF Protection** - State parameter validation in OAuth flow
- **HTTP-only Cookies** - Secure session management
- **Token Encryption** - Slack tokens stored securely
- **Input Validation** - Zod schemas for all API inputs
- **Rate Limiting** - Built-in Express rate limiting

## 📊 Features in Detail

### Message Composition
- **Character limit validation** (4,000 chars)
- **Real-time character count**
- **Channel selection with search**
- **Message preview**
- **Draft saving**

### Scheduling System
- **Timezone support** (UTC, EST, PST, GMT)
- **Future date validation**
- **Recurring messages** (planned)
- **Delivery status tracking**
- **Cancellation before delivery**

### Dashboard Analytics
- **Messages sent counter**
- **Scheduled messages counter**
- **Active channels counter**
- **Real-time updates**
- **Historical data** (planned)

## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Commit your changes** (`git commit -m 'Add amazing feature'`)
4. **Push to the branch** (`git push origin feature/amazing-feature`)
5. **Open a Pull Request**

### Development Guidelines
- Follow TypeScript best practices
- Add proper error handling
- Write meaningful commit messages
- Update documentation for new features
- Test OAuth flow thoroughly

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Issues**: [GitHub Issues](https://github.com/beginner1214/SlackFlow/issues)
- **Discussions**: [GitHub Discussions](https://github.com/beginner1214/SlackFlow/discussions)
- **Slack API Docs**: [api.slack.com](https://api.slack.com/)
- **ngrok Docs**: [ngrok.com/docs](https://ngrok.com/docs/)

## 🙏 Acknowledgments

- Built with [Slack Web API](https://api.slack.com/)
- UI components from [shadcn/ui](https://ui.shadcn.com/)
- Icons from [Font Awesome](https://fontawesome.com/)
- Local development tunneling via [ngrok](https://ngrok.com/)
- State management with [TanStack Query](https://tanstack.com/query)

## 🚀 Deployment

### Production Deployment
1. **Replace ngrok** with a proper domain and HTTPS certificate
2. **Update environment variables** for production
3. **Configure database** (PostgreSQL recommended for production)
4. **Set up proper logging** and monitoring
5. **Update Slack app** redirect URLs to production domain

### Environment Variables for Production
```env
SLACK_CLIENT_ID=your_slack_client_id
SLACK_CLIENT_SECRET=your_slack_client_secret
SLACK_REDIRECT_URI=https://yourdomain.com/oauth/callback
NODE_ENV=production
PORT=5000
DATABASE_URL=your_production_database_url
```

***

**Made with ❤️ for better Slack workflows**

**⭐ Star this repository if SlackFlow helps you manage your Slack messages more efficiently!**

The new sections provide insight into the technical architecture, design decisions, and the real challenges faced during development, especially around Slack's OAuth complexity and scope management.

[1](https://docs.slack.dev/authentication/installing-with-oauth)
[2](https://docs.slack.dev/authentication/best-practices-for-security)
[3](https://knock.app/blog/how-to-authenticate-users-in-slack-using-oauth)
[4](https://www.useparagon.com/blog/create-a-slack-app-to-enable-oauth-for-your-integration)
[5](https://moldstud.com/articles/p-essential-oauth-concepts-for-slack-developers-guide)
[6](https://workos.com/docs/integrations/slack-oauth)
[7](https://success.skyhighsecurity.com/Skyhigh_CASB/06_Skyhigh_CASB_Sanctioned_Applications/01_Skyhigh_CASB_Native_Sanctioned_Apps/Skyhigh_CASB_for_Slack_Non-Enterprise/Step_2:_Create_a_Custom_OAuth_App_in_Slack)
[8](https://api.slack.com/tutorials/tags/auth)
[9](https://community.appsmith.com/tutorial/slack-api-oauth2-integration-tutorial)
[10](https://www.magicbell.com/blog/build-slack-apps-for-collaboration)







## 📋 Overview

SlackFlow is a modern web application that streamlines Slack messaging workflows. It provides an intuitive interface for composing, scheduling, and managing Slack messages across multiple channels, with powerful OAuth integration and real-time synchronization.

### ✨ Key Features

- 🔐 **Secure Slack OAuth Integration** - Seamless authentication with your Slack workspace
- 📝 **Message Composition** - Rich text editor with character count and validation
- ⏰ **Message Scheduling** - Schedule messages for future delivery with timezone support
- 📊 **Analytics Dashboard** - Track sent messages, scheduled messages, and active channels
- 🚀 **Real-time Updates** - Live synchronization with Slack workspace data
- 🎨 **Modern UI/UX** - Clean, responsive design built with shadcn/ui components
- 📱 **Mobile Responsive** - Works seamlessly across all device sizes
- 🔄 **Message Management** - Cancel, reschedule, and monitor message delivery status

## 🛠️ Tech Stack

### Frontend
- **React 18** with TypeScript
- **Wouter** for client-side routing
- **TanStack Query** for state management and data fetching
- **React Hook Form** with Zod validation
- **shadcn/ui** components with Tailwind CSS
- **Vite** for development and building

### Backend
- **Node.js** with Express.js
- **TypeScript** for type safety
- **Slack Web API** for Slack integration
- **SQLite** database with custom storage layer
- **Cookie-based session management**
- **CSRF protection** with state validation

## 📋 Prerequisites

- **Node.js 18.x** or higher
- **npm** or yarn package manager
- **ngrok** installed globally (`npm install -g ngrok`)
- A **Slack workspace** with admin permissions
- **Slack app credentials** (Client ID, Client Secret)

## 🚀 Installation

### 1. Clone the Repository
```bash
git clone https://github.com/beginner1214/SlackFlow.git
cd SlackFlow
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Install and Set Up ngrokHere's the updated README with the **Architectural Overview** and **Challenges & Learnings** sections added:

# SlackFlow

> A powerful Slack messaging and scheduling application built with React and Node.js

![TypeScript](https://img.shields.io/badge/typescript-%23007ACC.svg?style=for-the-badge&logo=typescript&logohttps://img.shields.io/badge/react-%2320232a.svg?style=for-the-badge&logo=react&logoColor=%2361img.shields.io/badge/express.js-%23404d59.svg?style=for-the-badge&logo=express&logoColor=%2361img.shields.io/badge/Slack-4A154B?style=for-the-badge&logo=slack&logoColor=white.shields.io/badge/node.js-6DA55F?style=for-the-badge&







## 📋 Overview

SlackFlow is a modern web application that streamlines Slack messaging workflows. It provides an intuitive interface for composing, scheduling, and managing Slack messages across multiple channels, with powerful OAuth integration and real-time synchronization.

### ✨ Key Features

- 🔐 **Secure Slack OAuth Integration** - Seamless authentication with your Slack workspace
- 📝 **Message Composition** - Rich text editor with character count and validation
- ⏰ **Message Scheduling** - Schedule messages for future delivery with timezone support
- 📊 **Analytics Dashboard** - Track sent messages, scheduled messages, and active channels
- 🚀 **Real-time Updates** - Live synchronization with Slack workspace data
- 🎨 **Modern UI/UX** - Clean, responsive design built with shadcn/ui components
- 📱 **Mobile Responsive** - Works seamlessly across all device sizes
- 🔄 **Message Management** - Cancel, reschedule, and monitor message delivery status

## 🏗️ Architectural Overview

SlackFlow follows a **full-stack TypeScript architecture** with clear separation between client and server responsibilities:

### OAuth & Authentication Flow
```
User → SlackFlow → Slack OAuth → Authorization → Token Exchange → Secure Storage
```

1. **OAuth Initiation**: Client requests authorization URL from server
2. **Slack Authorization**: User redirects to Slack for workspace permission
3. **Callback Handling**: Server receives authorization code with CSRF protection
4. **Token Exchange**: Server exchanges code for Bot User OAuth Token (`xoxb-`)
5. **Secure Storage**: Tokens encrypted and stored with team/user mapping

### Token Management Strategy
- **Bot Tokens**: Stored securely in SQLite with team/user relationships
- **Session Management**: HTTP-only cookies with CSRF state validation
- **Token Validation**: Automatic token health checks every 30 seconds
- **Graceful Expiry**: Handles token expiration with user-friendly re-authentication

### Scheduled Task Architecture
```
React Scheduler → API Validation → Database Queue → Background Service → Slack API
```

- **Message Queuing**: SQLite-based persistent queue for scheduled messages
- **Background Processing**: Node.js `setInterval` checks for due messages every 60 seconds
- **Timezone Handling**: UTC storage with client-side timezone conversion
- **Delivery Tracking**: Status updates (pending → sent → failed) with retry logic
- **Cancellation Support**: Users can cancel messages before delivery

### API Design Pattern
- **RESTful Endpoints**: Clear resource-based URL structure
- **Middleware Chain**: Authentication → Validation → Business Logic → Response
- **Error Handling**: Comprehensive error catching with user-friendly messages
- **Type Safety**: End-to-end TypeScript with Zod schema validation

## 🧩 Challenges & Learnings

### Major Development Hurdles

#### 1. **Slack OAuth Scope Complexity** 🎯
**Challenge**: The biggest hurdle was navigating Slack's extensive documentation to understand the distinction between user tokens, bot tokens, and the specific scopes required for each operation.

**Key Confusions**:
- Difference between `chat:write` vs `chat:write.public` vs deprecated `chat:write:bot`
- When to use Bot Token Scopes vs User Token Scopes
- Understanding Enterprise Grid vs regular workspace limitations
- OAuth v2 vs legacy OAuth flows

**Solution**:
- Systematically tested each scope combination in a development workspace
- Created a scope mapping document for different use cases
- Implemented proper error handling to surface permission issues clearly
- Used Slack's OAuth test tool to validate token permissions

#### 2. **HTTPS Requirement in Development** 🔒
**Challenge**: Slack requires HTTPS URLs for OAuth redirects, but local development runs on HTTP.

**Solution**: 
- Integrated ngrok for local HTTPS tunneling
- Automated ngrok URL updates in development workflow
- Created clear documentation for developers to handle URL changes

#### 3. **Bot Channel Membership Management** 🤖
**Challenge**: Understanding when and how bots need to be invited to channels, even with appropriate scopes.

**Learnings**:
- `chat:write.public` doesn't eliminate the need for channel membership in all cases
- Private channels always require explicit bot invitation
- Workspace admin settings can override scope permissions

#### 4. **Response Stream Consumption Issues** 🔄
**Challenge**: Fetch API response bodies can only be read once, causing "stream already read" errors in error handling.

**Solution**:
- Implemented `Response.clone()` for multiple reads
- Restructured error handling to read once and parse appropriately
- Added proper error boundary patterns

#### 5. **State Management for OAuth Security** 🛡️
**Challenge**: Implementing CSRF protection with state parameter validation while maintaining user experience.

**Solution**:
- HTTP-only cookies for state storage
- Automatic cleanup of expired state tokens
- Clear error messages for state mismatch scenarios

### Key Technical Learnings

#### Slack API Best Practices
```typescript
// Always use Bot User OAuth Tokens (xoxb-) for API calls
const client = new WebClient(process.env.SLACK_BOT_TOKEN);

// Handle rate limiting gracefully
try {
  const result = await client.chat.postMessage({
    channel: channelId,
    text: message
  });
} catch (error) {
  if (error.code === 'rate_limited') {
    // Implement exponential backoff
    await delay(error.retryAfter * 1000);
    // Retry logic here
  }
}
```

#### Token Security Patterns
- Never expose tokens in client-side code
- Use environment variables for sensitive credentials
- Implement token rotation for production environments
- Regular token health checks to catch expired credentials

#### Database Design Insights
- Store team/user relationships for multi-workspace support
- Use UTC timestamps for all scheduling operations
- Implement soft deletes for message history tracking
- Index frequently queried fields (teamId, userId, scheduledFor)

### Development Process Improvements

#### Documentation-First Approach
- Created comprehensive API documentation before implementation
- Maintained a troubleshooting guide throughout development
- Documented all Slack scope requirements and use cases

#### Testing Strategy
- Manual testing with multiple Slack workspaces
- Edge case testing (expired tokens, deleted channels, offline scenarios)
- OAuth flow testing with different user permission levels

#### Error Handling Philosophy
- User-friendly error messages that suggest solutions
- Comprehensive logging for debugging without exposing sensitive data
- Graceful degradation when Slack API is unavailable

These challenges taught valuable lessons about third-party API integration, OAuth security, and building resilient real-time applications. The complexity of Slack's permission system, while initially overwhelming, ultimately led to a deeper understanding of enterprise-grade API design patterns.

## 🛠️ Tech Stack

### Frontend
- **React 18** with TypeScript
- **Wouter** for client-side routing
- **TanStack Query** for state management and data fetching
- **React Hook Form** with Zod validation
- **shadcn/ui** components with Tailwind CSS
- **Vite** for development and building

### Backend
- **Node.js** with Express.js
- **TypeScript** for type safety
- **Slack Web API** for Slack integration
- **SQLite** database with custom storage layer
- **Cookie-based session management**
- **CSRF protection** with state validation

## 📋 Prerequisites

- **Node.js 18.x** or higher
- **npm** or yarn package manager
- **ngrok** installed globally (`npm install -g ngrok`)
- A **Slack workspace** with admin permissions
- **Slack app credentials** (Client ID, Client Secret)

## 🚀 Installation

### 1. Clone the Repository
```bash
git clone https://github.com/beginner1214/SlackFlow.git
cd SlackFlow
```

### 2. Install Dependencies
```bash
npm install
```

### 3. Install and Set Up ngrok
```bash
# Install ngrok globally if not already installed
npm install -g ngrok

# Start ngrok to expose your local server
ngrok http 5000
```

**Important:** Copy the **HTTPS URL** from ngrok output (e.g., `https://abc123.ngrok-free.app`)

### 4. Environment Configuration

Create a `.env` file in the root directory with your ngrok HTTPS URL:

```env
# Slack App Credentials
SLACK_CLIENT_ID=your_slack_client_id
SLACK_CLIENT_SECRET=your_slack_client_secret

# IMPORTANT: Use your ngrok HTTPS URL here
SLACK_REDIRECT_URI=https://abc123.ngrok-free.app/oauth/callback

# Server Configuration
NODE_ENV=development
PORT=5000

# Database (optional - defaults to SQLite)
DATABASE_URL=./data/slackflow.db
```

### 5. Slack App Setup

1. **Create a Slack App**
   - Go to [api.slack.com/apps](https://api.slack.com/apps)
   - Click **"Create New App"** → **"From scratch"**
   - Enter app name (e.g., "SlackFlow") and select your workspace

2. **Configure OAuth & Permissions**
   - Navigate to **"OAuth & Permissions"** in the left sidebar
   - Add **Redirect URLs** (use your ngrok HTTPS URL):
     ```
     https://abc123.ngrok-free.app/oauth/callback
     ```

3. **Add Bot Token Scopes** (Essential):
   - `chat:write` - Send messages as the app
   - `chat:write.public` - Send messages to channels app isn't a member of
   - `channels:read` - View basic information about public channels
   - `channels:join` - Join public channels in a workspace

4. **Install App to Workspace**
   - Scroll up to **"OAuth Tokens for Your Workspace"**
   - Click **"Install to Workspace"**
   - Authorize the app
   - Copy the **Bot User OAuth Token** (starts with `xoxb-`)

5. **Update Environment Variables**
   - Add your Slack credentials to the `.env` file

### 6. Start Development Server

**Two Terminal Setup:**

**Terminal 1 - Keep ngrok running:**
```bash
ngrok http 5000
```

**Terminal 2 - Start the application:**
```bash
npm run dev
```

The application will be available at your **ngrok HTTPS URL** (e.g., `https://abc123.ngrok-free.app`)

## 🔧 Development Workflow

### Every Development Session:
1. **Start ngrok**: `ngrok http 5000`
2. **Copy the HTTPS URL** from ngrok output
3. **Update your `.env` file** with the new ngrok URL (if changed)
4. **Update Slack app redirect URLs** if the ngrok URL changed
5. **Start the dev server**: `npm run dev`

### Important Notes:
- **ngrok URLs change** each time you restart ngrok (unless using a paid static domain)
- **Always use HTTPS URLs** from ngrok, never HTTP
- **Update both** your `.env` file and Slack app settings when ngrok URL changes

## 📖 Usage

### Authentication
1. Open the application at your ngrok HTTPS URL
2. Click **"Connect to Slack"**
3. Authorize the app in your Slack workspace
4. You'll be redirected back to the dashboard

### Composing Messages
1. **Select a channel** from the dropdown
2. **Type your message** (max 4,000 characters)
3. **Choose delivery method:**
   - Send immediately
   - Schedule for later
4. Click **"Send Now"** or **"Schedule Message"**

### Scheduling Messages
1. Check **"Schedule message for later"**
2. Select **date and time**
3. Choose **timezone**
4. Click **"Schedule Message"**

### Managing Scheduled Messages
- View all scheduled messages in the **"Scheduled"** tab
- **Cancel pending messages** before they're sent
- **Monitor delivery status** and history

### Inviting Bot to Channels
If you get a "Bot not in channel" error:
```bash
# In Slack, type in the target channel:
/invite @YourBotName
```

## 🔧 API Endpoints

### Authentication
- `GET /api/slack/auth` - Get Slack OAuth authorization URL
- `POST /api/slack/oauth/callback` - Handle OAuth callback
- `GET /api/slack/status/:teamId/:userId` - Check connection status
- `POST /api/slack/disconnect/:teamId/:userId` - Disconnect Slack account

### Messaging
- `GET /api/slack/channels/:teamId/:userId` - List available channels
- `POST /api/slack/send/:teamId/:userId` - Send immediate message
- `POST /api/messages/schedule/:teamId/:userId` - Schedule message for later

### Message Management
- `GET /api/messages/scheduled/:teamId/:userId` - Get scheduled messages
- `DELETE /api/messages/scheduled/:teamId/:userId/:id` - Cancel scheduled message
- `GET /api/messages/stats/:teamId/:userId` - Get messaging statistics

## 🚨 Troubleshooting

### Common Issues

**"Redirect URI Mismatch" Error**
```bash
# Ensure your .env SLACK_REDIRECT_URI matches your Slack app settings
# Both should use the same ngrok HTTPS URL
SLACK_REDIRECT_URI=https://your-ngrok-url.ngrok-free.app/oauth/callback
```

**"Bot not in channel or insufficient permissions" Error**
```bash
# Invite your bot to the channel in Slack
/invite @YourBotName

# Or add via channel settings:
# Channel → Settings → Integrations → Add apps
```

**ngrok URL Changed**
1. Copy the new ngrok HTTPS URL
2. Update `.env` file with new URL
3. Update Slack app redirect URLs in [api.slack.com/apps](https://api.slack.com/apps)
4. Restart your development server: `npm run dev`

**"404 Page Not Found" Error**
- Make sure you're accessing the **ngrok HTTPS URL**, not `localhost:5000`
- Verify ngrok is properly forwarding to `127.0.0.1:5000`

**SelectItem Empty Value Error**
- Check for any `<SelectItem value="">` components
- Replace with non-empty values like `value="none"`

**"Failed to execute 'text' on 'Response': body stream already read"**
- This indicates a response parsing issue in the error handler
- Restart the development server

**OAuth State Parameter Errors**
- Ensure cookies are enabled in your browser
- Clear browser cookies for the ngrok domain
- Restart the development server

## 🗂️ Project Structure

```
SlackFlow/
├── client/                     # React frontend
│   ├── src/
│   │   ├── components/         # UI components
│   │   │   ├── ui/            # shadcn/ui components
│   │   │   ├── header.tsx     # App header
│   │   │   ├── sidebar.tsx    # Navigation sidebar
│   │   │   ├── compose-section.tsx    # Message composition
│   │   │   └── scheduled-messages.tsx # Scheduled messages view
│   │   ├── hooks/             # Custom React hooks
│   │   │   ├── use-auth.ts    # Authentication hook
│   │   │   └── use-toast.ts   # Toast notifications
│   │   ├── lib/               # Utilities and configurations
│   │   │   ├── queryClient.ts # TanStack Query setup
│   │   │   └── utils.ts       # Utility functions
│   │   ├── pages/             # Route components
│   │   │   ├── dashboard.tsx  # Main dashboard
│   │   │   ├── oauth-callback.tsx # OAuth handler
│   │   │   └── not-found.tsx  # 404 page
│   │   └── main.tsx           # Entry point
│   ├── index.html             # HTML template
│   └── vite.config.ts         # Vite configuration
├── server/                    # Express backend
│   ├── services/              # Business logic
│   │   ├── slack.ts          # Slack API integration
│   │   └── scheduler.ts      # Message scheduling
│   ├── shared/               # Shared types and schemas
│   │   └── schema.ts        # Validation schemas
│   ├── routes.ts            # API routes
│   ├── storage.ts           # Database layer
│   ├── vite.ts              # Vite SSR setup
│   └── index.ts             # Server entry point
├── db/                      # Database files
├── .env                     # Environment variables
├── package.json            # Dependencies
└── README.md              # This file
```

## 🔒 Security Features

- **CSRF Protection** - State parameter validation in OAuth flow
- **HTTP-only Cookies** - Secure session management
- **Token Encryption** - Slack tokens stored securely
- **Input Validation** - Zod schemas for all API inputs
- **Rate Limiting** - Built-in Express rate limiting

## 📊 Features in Detail

### Message Composition
- **Character limit validation** (4,000 chars)
- **Real-time character count**
- **Channel selection with search**
- **Message preview**
- **Draft saving**

### Scheduling System
- **Timezone support** (UTC, EST, PST, GMT)
- **Future date validation**
- **Recurring messages** (planned)
- **Delivery status tracking**
- **Cancellation before delivery**

### Dashboard Analytics
- **Messages sent counter**
- **Scheduled messages counter**
- **Active channels counter**
- **Real-time updates**
- **Historical data** (planned)

## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Commit your changes** (`git commit -m 'Add amazing feature'`)
4. **Push to the branch** (`git push origin feature/amazing-feature`)
5. **Open a Pull Request**

### Development Guidelines
- Follow TypeScript best practices
- Add proper error handling
- Write meaningful commit messages
- Update documentation for new features
- Test OAuth flow thoroughly

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Issues**: [GitHub Issues](https://github.com/beginner1214/SlackFlow/issues)
- **Discussions**: [GitHub Discussions](https://github.com/beginner1214/SlackFlow/discussions)
- **Slack API Docs**: [api.slack.com](https://api.slack.com/)
- **ngrok Docs**: [ngrok.com/docs](https://ngrok.com/docs/)

## 🙏 Acknowledgments

- Built with [Slack Web API](https://api.slack.com/)
- UI components from [shadcn/ui](https://ui.shadcn.com/)
- Icons from [Font Awesome](https://fontawesome.com/)
- Local development tunneling via [ngrok](https://ngrok.com/)
- State management with [TanStack Query](https://tanstack.com/query)

## 🚀 Deployment

### Production Deployment
1. **Replace ngrok** with a proper domain and HTTPS certificate
2. **Update environment variables** for production
3. **Configure database** (PostgreSQL recommended for production)
4. **Set up proper logging** and monitoring
5. **Update Slack app** redirect URLs to production domain

### Environment Variables for Production
```env
SLACK_CLIENT_ID=your_slack_client_id
SLACK_CLIENT_SECRET=your_slack_client_secret
SLACK_REDIRECT_URI=https://yourdomain.com/oauth/callback
NODE_ENV=production
PORT=5000
DATABASE_URL=your_production_database_url
```

***

**Made with ❤️ for better Slack workflows**

**⭐ Star this repository if SlackFlow helps you manage your Slack messages more efficiently!**

The new sections provide insight into the technical architecture, design decisions, and the real challenges faced during development, especially around Slack's OAuth complexity and scope management.

[1](https://docs.slack.dev/authentication/installing-with-oauth)
[2](https://docs.slack.dev/authentication/best-practices-for-security)
[3](https://knock.app/blog/how-to-authenticate-users-in-slack-using-oauth)
[4](https://www.useparagon.com/blog/create-a-slack-app-to-enable-oauth-for-your-integration)
[5](https://moldstud.com/articles/p-essential-oauth-concepts-for-slack-developers-guide)
[6](https://workos.com/docs/integrations/slack-oauth)
[7](https://success.skyhighsecurity.com/Skyhigh_CASB/06_Skyhigh_CASB_Sanctioned_Applications/01_Skyhigh_CASB_Native_Sanctioned_Apps/Skyhigh_CASB_for_Slack_Non-Enterprise/Step_2:_Create_a_Custom_OAuth_App_in_Slack)
[8](https://api.slack.com/tutorials/tags/auth)
[9](https://community.appsmith.com/tutorial/slack-api-oauth2-integration-tutorial)
[10](https://www.magicbell.com/blog/build-slack-apps-for-collaboration)
### Inviting Bot to Channels
If you get a "Bot not in channel" error:
```bash
# In Slack, type in the target channel:
/invite @YourBotName
```

## 🔧 API Endpoints

### Authentication
- `GET /api/slack/auth` - Get Slack OAuth authorization URL
- `POST /api/slack/oauth/callback` - Handle OAuth callback
- `GET /api/slack/status/:teamId/:userId` - Check connection status
- `POST /api/slack/disconnect/:teamId/:userId` - Disconnect Slack account

### Messaging
- `GET /api/slack/channels/:teamId/:userId` - List available channels
- `POST /api/slack/send/:teamId/:userId` - Send immediate message
- `POST /api/messages/schedule/:teamId/:userId` - Schedule message for later

### Message Management
- `GET /api/messages/scheduled/:teamId/:userId` - Get scheduled messages
- `DELETE /api/messages/scheduled/:teamId/:userId/:id` - Cancel scheduled message
- `GET /api/messages/stats/:teamId/:userId` - Get messaging statistics

## 🚨 Troubleshooting

### Common Issues

**"Redirect URI Mismatch" Error**
```bash
# Ensure your .env SLACK_REDIRECT_URI matches your Slack app settings
# Both should use the same ngrok HTTPS URL
SLACK_REDIRECT_URI=https://your-ngrok-url.ngrok-free.app/oauth/callback
```

**"Bot not in channel or insufficient permissions" Error**
```bash
# Invite your bot to the channel in Slack
/invite @YourBotName

# Or add via channel settings:
# Channel → Settings → Integrations → Add apps
```

**ngrok URL Changed**
1. Copy the new ngrok HTTPS URL
2. Update `.env` file with new URL
3. Update Slack app redirect URLs in [api.slack.com/apps](https://api.slack.com/apps)
4. Restart your development server: `npm run dev`

**"404 Page Not Found" Error**
- Make sure you're accessing the **ngrok HTTPS URL**, not `localhost:5000`
- Verify ngrok is properly forwarding to `127.0.0.1:5000`

**SelectItem Empty Value Error**
- Check for any `<SelectItem value="">` components
- Replace with non-empty values like `value="none"`

**"Failed to execute 'text' on 'Response': body stream already read"**
- This indicates a response parsing issue in the error handler
- Restart the development server

**OAuth State Parameter Errors**
- Ensure cookies are enabled in your browser
- Clear browser cookies for the ngrok domain
- Restart the development server

## 🗂️ Project Structure

```
SlackFlow/
├── client/                     # React frontend
│   ├── src/
│   │   ├── components/         # UI components
│   │   │   ├── ui/            # shadcn/ui components
│   │   │   ├── header.tsx     # App header
│   │   │   ├── sidebar.tsx    # Navigation sidebar
│   │   │   ├── compose-section.tsx    # Message composition
│   │   │   └── scheduled-messages.tsx # Scheduled messages view
│   │   ├── hooks/             # Custom React hooks
│   │   │   ├── use-auth.ts    # Authentication hook
│   │   │   └── use-toast.ts   # Toast notifications
│   │   ├── lib/               # Utilities and configurations
│   │   │   ├── queryClient.ts # TanStack Query setup
│   │   │   └── utils.ts       # Utility functions
│   │   ├── pages/             # Route components
│   │   │   ├── dashboard.tsx  # Main dashboard
│   │   │   ├── oauth-callback.tsx # OAuth handler
│   │   │   └── not-found.tsx  # 404 page
│   │   └── main.tsx           # Entry point
│   ├── index.html             # HTML template
│   └── vite.config.ts         # Vite configuration
├── server/                    # Express backend
│   ├── services/              # Business logic
│   │   ├── slack.ts          # Slack API integration
│   │   └── scheduler.ts      # Message scheduling
│   ├── shared/               # Shared types and schemas
│   │   └── schema.ts        # Validation schemas
│   ├── routes.ts            # API routes
│   ├── storage.ts           # Database layer
│   ├── vite.ts              # Vite SSR setup
│   └── index.ts             # Server entry point
├── db/                      # Database files
├── .env                     # Environment variables
├── package.json            # Dependencies
└── README.md              # This file
```

## 🔒 Security Features

- **CSRF Protection** - State parameter validation in OAuth flow
- **HTTP-only Cookies** - Secure session management
- **Token Encryption** - Slack tokens stored securely
- **Input Validation** - Zod schemas for all API inputs
- **Rate Limiting** - Built-in Express rate limiting

## 📊 Features in Detail

### Message Composition
- **Character limit validation** (4,000 chars)
- **Real-time character count**
- **Channel selection with search**
- **Message preview**
- **Draft saving**

### Scheduling System
- **Timezone support** (UTC, EST, PST, GMT)
- **Future date validation**
- **Recurring messages** (planned)
- **Delivery status tracking**
- **Cancellation before delivery**

### Dashboard Analytics
- **Messages sent counter**
- **Scheduled messages counter**
- **Active channels counter**
- **Real-time updates**
- **Historical data** (planned)

## 🤝 Contributing

1. **Fork the repository**
2. **Create a feature branch** (`git checkout -b feature/amazing-feature`)
3. **Commit your changes** (`git commit -m 'Add amazing feature'`)
4. **Push to the branch** (`git push origin feature/amazing-feature`)
5. **Open a Pull Request**

### Development Guidelines
- Follow TypeScript best practices
- Add proper error handling
- Write meaningful commit messages
- Update documentation for new features
- Test OAuth flow thoroughly

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🆘 Support

- **Issues**: [GitHub Issues](https://github.com/beginner1214/SlackFlow/issues)
- **Discussions**: [GitHub Discussions](https://github.com/beginner1214/SlackFlow/discussions)
- **Slack API Docs**: [api.slack.com](https://api.slack.com/)
- **ngrok Docs**: [ngrok.com/docs](https://ngrok.com/docs/)

## 🙏 Acknowledgments

- Built with [Slack Web API](https://api.slack.com/)
- UI components from [shadcn/ui](https://ui.shadcn.com/)
- Icons from [Font Awesome](https://fontawesome.com/)
- Local development tunneling via [ngrok](https://ngrok.com/)
- State management with [TanStack Query](https://tanstack.com/query)

## 🚀 Deployment

### Production Deployment
1. **Replace ngrok** with a proper domain and HTTPS certificate
2. **Update environment variables** for production
3. **Configure database** (PostgreSQL recommended for production)
4. **Set up proper logging** and monitoring
5. **Update Slack app** redirect URLs to production domain

### Environment Variables for Production
```env
SLACK_CLIENT_ID=your_slack_client_id
SLACK_CLIENT_SECRET=your_slack_client_secret
SLACK_REDIRECT_URI=https://yourdomain.com/oauth/callback
NODE_ENV=production
PORT=5000
DATABASE_URL=your_production_database_url
```

***

**Made with ❤️ for better Slack workflows**

**⭐ Star this repository if SlackFlow helps you manage your Slack messages more efficiently!**

[1](https://ngrok.com/docs/)
[2](https://ngrok.com/docs/getting-started/)
[3](https://ngrok.com/docs/k8s/guides/local-cluster/)
[4](https://docs.developer.disruptive-technologies.com/data-connectors/development-guides/local-development-with-ngrok)
[5](https://develop.sentry.dev/development-infrastructure/ngrok/)
[6](https://ngrok.com/docs/universal-gateway/examples/secure-developer-environments/)
[7](https://ngrok.com/docs/integrations/hostedhooks/webhooks/)
[8](https://www.sitepoint.com/use-ngrok-test-local-site/)
