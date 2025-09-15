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

[1](https://ngrok.com/docs/)
[2](https://ngrok.com/docs/getting-started/)
[3](https://ngrok.com/docs/k8s/guides/local-cluster/)
[4](https://docs.developer.disruptive-technologies.com/data-connectors/development-guides/local-development-with-ngrok)
[5](https://develop.sentry.dev/development-infrastructure/ngrok/)
[6](https://ngrok.com/docs/universal-gateway/examples/secure-developer-environments/)
[7](https://ngrok.com/docs/integrations/hostedhooks/webhooks/)
[8](https://www.sitepoint.com/use-ngrok-test-local-site/)
