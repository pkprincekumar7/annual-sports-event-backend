# Annual Sports Event Backend API

Backend server for Annual Sports Events Registration Portal for Purnea College of Engineering, Purnea.

## Tech Stack

- **Express.js** - Backend server framework
- **Node.js** - Runtime environment
- **JWT** - Authentication tokens
- **XLSX** - Excel export functionality
- **CORS** - Cross-origin resource sharing

## Features

- ✅ JWT-based authentication
- ✅ Player registration and management
- ✅ Captain assignment and team management
- ✅ Team and individual event registration
- ✅ Participation tracking and limits (max 10 participations)
- ✅ Registration deadline enforcement
- ✅ Duplicate team name prevention per sport
- ✅ Excel export with team information
- ✅ RESTful API endpoints

## Prerequisites

- Node.js (v16 or higher)
- npm or yarn
- For remote deployment: EC2 instance or similar Linux server

## Environment Variables

Create a `.env` file in the root directory with the following variables:

```env
PORT=3001
JWT_SECRET=your-secret-key-change-in-production
REGISTRATION_DEADLINE=2026-01-07T00:00:00
```

- `PORT`: Server port (default: 3001)
- `JWT_SECRET`: Secret key for JWT token signing (change in production)
- `REGISTRATION_DEADLINE`: Registration deadline date in local timezone format (default: 2026-01-07T00:00:00)

## Local Development Setup

### 1. Install Dependencies

```bash
npm install
```

### 2. Configure Environment Variables

Create a `.env` file in the root directory:

```bash
# Create .env file manually with the variables above
```

### 3. Start the Server

```bash
npm run server
# or
npm start
```

The server will run on `http://localhost:3001` (or the port specified in `.env`)

### 4. Development Mode with Auto-reload

For development with auto-reload:

```bash
npm run dev
```

## Remote Instance Setup (EC2/Linux Server)

### 1. Connect to Your Server

```bash
ssh user@your-server-ip
```

### 2. Install Node.js (if not already installed)

```bash
# Using NodeSource repository (recommended)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

### 3. Clone and Setup Project

```bash
# Navigate to your desired directory
cd /opt  # or /var/www or your preferred location

# Clone your repository
git clone <your-repo-url> annual-sports-backend
cd annual-sports-backend

# Install dependencies
npm install --production
```

### 4. Create Environment Variables

```bash
# Create .env file
nano .env
```

Add the following content:

```env
PORT=3001
JWT_SECRET=your-strong-secret-key-here
REGISTRATION_DEADLINE=2026-01-07T00:00:00
```

Save and exit (Ctrl+X, then Y, then Enter)

### 5. Create Systemd Service File

Create a systemd service file to manage the application:

```bash
sudo nano /etc/systemd/system/annual-sports-backend.service
```

Add the following content (adjust paths as needed):

```ini
[Unit]
Description=Annual Sports Backend Server
After=network.target

[Service]
Type=simple
User=your-username
WorkingDirectory=/opt/annual-sports-backend
Environment="NODE_ENV=production"
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=10
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=annual-sports-backend

[Install]
WantedBy=multi-user.target
```

**Important:** Replace:
- `your-username` with your actual Linux username
- `/opt/annual-sports-backend` with your actual project path

### 6. Enable and Start the Service

```bash
# Reload systemd to recognize the new service
sudo systemctl daemon-reload

# Enable the service to start on boot
sudo systemctl enable annual-sports-backend

# Start the service
sudo systemctl start annual-sports-backend

# Check service status
sudo systemctl status annual-sports-backend
```

### 7. Useful Systemctl Commands

```bash
# Check service status
sudo systemctl status annual-sports-backend

# Start the service
sudo systemctl start annual-sports-backend

# Stop the service
sudo systemctl stop annual-sports-backend

# Restart the service
sudo systemctl restart annual-sports-backend

# View logs
sudo journalctl -u annual-sports-backend -f

# View recent logs
sudo journalctl -u annual-sports-backend -n 50
```

### 8. Configure Firewall (if needed)

```bash
# Allow port 3001 (or your configured port)
sudo ufw allow 3001/tcp

# Or if using iptables
sudo iptables -A INPUT -p tcp --dport 3001 -j ACCEPT
```

### 9. Setup Reverse Proxy (Optional but Recommended)

If you want to use a domain name or port 80/443, set up Nginx as a reverse proxy:

```bash
# Install Nginx
sudo apt-get update
sudo apt-get install nginx

# Create Nginx configuration
sudo nano /etc/nginx/sites-available/annual-sports-backend
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com;  # Replace with your domain or IP

    location / {
        proxy_pass http://localhost:3001;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/annual-sports-backend /etc/nginx/sites-enabled/
sudo nginx -t  # Test configuration
sudo systemctl restart nginx
```

## Project Structure

```
├── data/
│   └── json_store/
│       └── players.json          # Player data storage
├── server.js                      # Express.js backend server
├── package.json                   # Dependencies and scripts
├── .env                          # Environment variables (not in git)
├── .gitignore                    # Git ignore rules
├── README.md                      # This file
└── SERVER_README.md              # Detailed API documentation
```

## Data Storage

- Player data is stored in `data/json_store/players.json`
- The JSON file is automatically created if it doesn't exist
- All player information, participations, and captain assignments are stored locally
- The directory structure is automatically created on server startup

## API Endpoints Overview

### Authentication
- `POST /api/login` - User login (returns JWT token)

### Player Management
- `GET /api/players` - Get all players (requires authentication)
- `POST /api/save-player` - Register a new player
- `POST /api/save-players` - Register multiple players (team events)
- `PUT /api/update-player` - Update player data (admin only)

### Captain Management
- `GET /api/sports` - Get list of team sports (admin only)
- `GET /api/captains-by-sport` - Get captains grouped by sport (admin only)
- `POST /api/add-captain` - Assign captain role (admin only)
- `DELETE /api/remove-captain` - Remove captain role (admin only)

### Participation Management
- `POST /api/validate-participations` - Validate team participation before registration
- `POST /api/update-team-participation` - Register/update team participation
- `POST /api/update-participation` - Register for individual events
- `DELETE /api/remove-participation` - Remove participation (admin only)

### Team Management
- `GET /api/teams/:sport` - Get all teams for a sport
- `POST /api/update-team-player` - Replace a player in a team (admin only)
- `DELETE /api/delete-team` - Delete a team (admin only)

### Participants
- `GET /api/participants/:sport` - Get all participants for a sport (admin only)

### Export
- `GET /api/export-excel` - Export players data to Excel (admin only)

For detailed API documentation, see [SERVER_README.md](./SERVER_README.md)

## Registration Deadline

The system enforces a registration deadline configured via `REGISTRATION_DEADLINE` environment variable. After the deadline:

- ✅ GET requests are allowed (read-only operations)
- ✅ Login is allowed
- ❌ All other write operations (POST, PUT, PATCH, DELETE) are blocked

The deadline check uses the server's local timezone.

## Team Events

The following sports are team events (require team registration):
- Cricket
- Volleyball
- Badminton
- Table Tennis
- Kabaddi
- Relay 4×100 m
- Relay 4×400 m

All other sports are individual events.

## Security Features

- JWT-based authentication
- Password stored as plain text (consider hashing in production)
- CORS configured to allow all origins
- Registration deadline enforcement
- Duplicate team name prevention per sport
- Admin-only endpoints protected
- Token expiration (24 hours)

## Admin Access

- Admin user registration number: `admin`
- Admin can manage all players, teams, and captains
- Admin can export data to Excel

## CORS Configuration

The server is configured to allow all origins (`origin: '*'`). Allowed methods:
- GET
- POST
- PUT
- PATCH
- DELETE
- OPTIONS

## Troubleshooting

### Service won't start
- Check logs: `sudo journalctl -u annual-sports-backend -n 50`
- Verify `.env` file exists and has correct values
- Check file permissions on `data/json_store/` directory
- Verify Node.js is installed: `node --version`

### Port already in use
- Change `PORT` in `.env` file
- Or stop the process using the port: `sudo lsof -i :3001`

### Permission denied errors
- Ensure the service user has read/write access to the project directory
- Check `data/json_store/` directory permissions: `chmod -R 755 data/`

### API returns 400 after registration deadline
- This is expected behavior - only GET requests and login are allowed after the deadline
- Update `REGISTRATION_DEADLINE` in `.env` if you need to extend registration

## API Response Format

All API responses follow this format:

**Success:**
```json
{
  "success": true,
  "message": "Operation successful",
  "data": { ... }
}
```

**Error:**
```json
{
  "success": false,
  "error": "Error message here"
}
```

## Authentication

Most endpoints require JWT authentication. Include the token in the Authorization header:

```
Authorization: Bearer <your-jwt-token>
```

The token is obtained from the `/api/login` endpoint and expires after 24 hours.

## Notes

- The server stores data in JSON format (consider database migration for production)
- JWT tokens expire after 24 hours
- Maximum 10 participations per player (including captain roles)
- Team names must be unique per sport (case-insensitive)
- Server timezone should be set to IST (Indian Standard Time) for correct deadline enforcement

## License

[Add your license information here]
