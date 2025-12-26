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
# Allow port 3001 (or your configured port) - only needed if not using Nginx
sudo ufw allow 3001/tcp

# If using Nginx with HTTPS, allow these ports instead:
sudo ufw allow 80/tcp   # HTTP (for Let's Encrypt verification)
sudo ufw allow 443/tcp  # HTTPS
sudo ufw allow 'Nginx Full'

# Or if using iptables
sudo iptables -A INPUT -p tcp --dport 3001 -j ACCEPT  # Only if not using Nginx
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT   # HTTP
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT  # HTTPS
```

**Note:** If you're using Nginx as a reverse proxy, you typically don't need to expose port 3001 publicly. Nginx will handle external traffic on ports 80/443 and forward to your Node.js app on localhost:3001.

### 9. Setup Reverse Proxy with HTTPS (Required for HTTPS Frontend)

**IMPORTANT:** If your frontend is hosted on HTTPS, your backend MUST also use HTTPS to avoid mixed content errors.

#### Step 1: Install Nginx

```bash
# Install Nginx
sudo apt-get update
sudo apt-get install nginx
```

#### Step 2: Install Certbot (Let's Encrypt SSL)

```bash
# Install Certbot
sudo apt-get install certbot python3-certbot-nginx
```

#### Step 3: Create Initial Nginx Configuration (HTTP)

```bash
# Create Nginx configuration
sudo nano /etc/nginx/sites-available/annual-sports-backend
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name your-domain.com;  # Replace with your actual domain name

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

#### Step 4: Obtain SSL Certificate

```bash
# Get SSL certificate (replace with your domain)
sudo certbot --nginx -d your-domain.com

# Or if you have multiple domains/subdomains:
sudo certbot --nginx -d your-domain.com -d www.your-domain.com
```

Certbot will automatically:
- Obtain SSL certificate from Let's Encrypt
- Update your Nginx configuration to use HTTPS
- Set up automatic certificate renewal

#### Step 5: Verify HTTPS Configuration

After Certbot runs, your Nginx config will be updated to include HTTPS. It should look like:

```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;  # Redirect HTTP to HTTPS
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    
    # SSL configuration (added by Certbot)
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;

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

#### Step 6: Update Firewall

```bash
# Allow HTTPS (port 443)
sudo ufw allow 443/tcp
sudo ufw allow 'Nginx Full'
```

#### Step 7: Test SSL Certificate Auto-Renewal

```bash
# Test renewal (dry run)
sudo certbot renew --dry-run
```

Certbot automatically renews certificates. You can verify the renewal timer:

```bash
sudo systemctl status certbot.timer
```

#### Alternative: If You Don't Have a Domain Name

If you're using an IP address instead of a domain:

1. **Option A: Use a self-signed certificate** (not recommended for production, browsers will show warnings):
   ```bash
   # Generate self-signed certificate
   sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
     -keyout /etc/nginx/ssl/nginx-selfsigned.key \
     -out /etc/nginx/ssl/nginx-selfsigned.crt
   ```

2. **Option B: Use a free domain** (recommended):
   - Get a free domain from services like Freenom, No-IP, or DuckDNS
   - Point it to your server's IP address
   - Follow the Certbot steps above

#### Update Frontend API URL

After setting up HTTPS, update your frontend to use the HTTPS backend URL:

```javascript
// Example: Update your API base URL
const API_BASE_URL = 'https://your-domain.com';
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

### HTTPS/Mixed Content Issues

**Problem:** Frontend on HTTPS cannot connect to backend on HTTP (Mixed Content Error)

**Solution:** Set up HTTPS for your backend using Nginx and Let's Encrypt (see Step 9 above).

**Quick Checklist:**
1. ✅ Backend has a domain name (not just IP address)
2. ✅ Nginx is installed and configured
3. ✅ SSL certificate is obtained (via Certbot)
4. ✅ Port 443 (HTTPS) is open in firewall
5. ✅ Frontend API URL is updated to use `https://` instead of `http://`

**Common Issues:**

- **"Mixed Content" error in browser console:**
  - Ensure backend URL in frontend uses `https://` protocol
  - Check that SSL certificate is valid and not expired

- **Certbot fails to obtain certificate:**
  - Ensure domain DNS points to your server IP
  - Ensure port 80 is open (needed for Let's Encrypt verification)
  - Check firewall rules: `sudo ufw status`

- **Nginx 502 Bad Gateway:**
  - Check if Node.js app is running: `sudo systemctl status annual-sports-backend`
  - Verify proxy_pass points to correct port: `http://localhost:3001`
  - Check Nginx error logs: `sudo tail -f /var/log/nginx/error.log`

- **SSL certificate expired:**
  - Certbot should auto-renew, but you can manually renew: `sudo certbot renew`
  - Check renewal status: `sudo certbot certificates`

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
