# Portainer Standalone Voice Stack - Simple Token Setup

## Quick Deploy with Registration Token

This is the simplest way to deploy your family voice stack with token-based registration.

### 1. Deploy in Portainer

1. **Go to Portainer** → **Stacks** → **Add Stack**
2. **Name**: `voice-stack`
3. **Repository URL**: `https://github.com/anykolaiszyn/voice-stack.git`
4. **Compose Path**: `docker-compose.yml`

### 2. Set Environment Variables

In the **Environment Variables** section, add:

| Variable | Value | Description |
|----------|-------|-------------|
| `SERVER_NAME` | `matrix.your-domain.com` | Your server domain |
| `REGISTRATION_TOKEN` | `family2024` | Token people need to register |
| `POSTGRES_PASSWORD` | `secure_db_password` | Database password |
| `REGISTRATION_SECRET` | `your_admin_secret` | Admin registration secret |
| `TURN_SECRET` | `your_turn_secret` | TURN server secret |

**Example Environment Variables:**
```
SERVER_NAME=matrix.byte-box.org
REGISTRATION_TOKEN=family2024
POSTGRES_PASSWORD=mySecureDbPass123
REGISTRATION_SECRET=myAdminSecret456
TURN_SECRET=myTurnSecret789
SYNAPSE_PORT=8008
ELEMENT_PORT=8080
TURN_PORT=3478
TURNS_PORT=5349
```

### 3. Deploy

Click **Deploy the Stack**

### 4. Access Your Server

- **Element Web**: `http://your-server-ip:8080`
- **Synapse API**: `http://your-server-ip:8008`

> **💡 Reverse Proxy Integration**: If you use Nginx Proxy Manager or another reverse proxy, see [`REVERSE-PROXY.md`](REVERSE-PROXY.md) for custom domain setup (e.g., `https://chat.your-domain.com`)

### 5. Register Users with Token

When someone wants to create an account:

1. Go to `http://your-server-ip:8080`
2. Click **"Create Account"**
3. Enter the registration token: `family2024` (or whatever you set)
4. Complete registration

## How It Works

### With Registration Token Set

- ✅ Registration **enabled** but requires token
- ✅ Only people with your token can register
- ✅ Perfect for family/friends
- ✅ No admin needed to create accounts

### Without Registration Token

- ❌ Registration **disabled** (family-safe mode)
- ✅ Only admin can create accounts manually
- ✅ Maximum security

## Token Security

### Good Token Examples:
- `family2024`
- `smith_family`
- `friends_welcome`
- `holiday_chat`

### Bad Token Examples:
- `123456` (too simple)
- `password` (too common)
- `token` (too obvious)

## Managing Users

### Create Admin Account (Optional)

If you need admin access for management:

```bash
# In Portainer → Containers → voice-stack-synapse → Console
register_new_matrix_user -c /data/homeserver.yaml -u admin -p AdminPass123! -a http://localhost:8008
```

### Change Registration Token

To change the token:

1. Go to Portainer → Stacks → voice-stack
2. Edit the stack
3. Change `REGISTRATION_TOKEN` value
4. Click **Update the Stack**

### Disable Token Registration

To disable new registrations:

1. Remove the `REGISTRATION_TOKEN` variable (or set it empty)
2. Update the stack
3. Only admin can now create accounts

## Troubleshooting

### Users Can't Register
- Check the `REGISTRATION_TOKEN` is set correctly
- Verify the token matches what users are entering
- Check container logs: Portainer → Containers → voice-stack-synapse → Logs

### Server Not Accessible
- Check port conflicts (8008, 8080)
- Verify firewall settings
- Check container health: Portainer → Containers

### Token Not Working
- Wait 30 seconds after deployment for token creation
- Check Synapse logs for token creation messages
- Try restarting the voice-stack-synapse container

## Resolving the Element Web “no default server specified” Loop

When Element Web logs keep repeating:

```
Configuring Element Web...
Invalid configuration: no default server specified
```

make sure your compose file’s Element Web service uses this startup snippet:

```yaml
  element-web:
    image: nginx:alpine
    container_name: voice-stack-element
    ports:
      - "${ELEMENT_PORT:-8080}:80"
    environment:
      - SYNAPSE_URL=http://synapse:${SYNAPSE_PORT:-8008}
      - SERVER_NAME=${SERVER_NAME:-matrix.byte-box.org}
    command: >
      /bin/sh -euc "
        echo 'Installing prerequisites...';
        apk add --no-cache wget tar;
        echo 'Downloading Element Web...';
        wget -qO- https://github.com/vector-im/element-web/releases/latest/download/element-web.tar.gz \
          | tar xz -C /usr/share/nginx/html;
        echo 'Generating config.json...';
        cat > /usr/share/nginx/html/config.json << 'EOF'
        {
          "default_server_config": {
            "m.homeserver": {
              "base_url": "${SYNAPSE_URL}",
              "server_name": "${SERVER_NAME}"
            }
          },
          "disable_custom_urls": false,
          "disable_guests": true
        }
        EOF
        echo 'Starting nginx...';
        exec nginx -g 'daemon off;'
      "
    networks:
      - voice-stack-network
      - proxy-bridge
```

## Perfect for Families

This setup gives you:

- 🔒 **Secure registration** with your custom token
- 👨‍👩‍👧‍👦 **Family-friendly** with no public room access
- 🚀 **Easy deployment** in Portainer
- 🔄 **Simple management** through environment variables
- 💬 **Private communication** for your family/friends

Just share your registration token with people you want to give access to!
