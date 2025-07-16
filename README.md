# WAHA Docker with Caddy

A production-ready deployment setup for [WAHA (WhatsApp HTTP API)](https://waha.devlike.pro/) using Docker Compose and Caddy as a reverse proxy with automatic SSL certificate management.

## Features

- 🚀 Easy deployment to cloud providers (Digital Ocean, Hetzner Cloud, etc.)
- 🔒 Automatic SSL certificates via Let's Encrypt
- 🔐 Optional Basic Authentication
- 🐳 Docker-based for easy management
- 🔄 Automatic container restarts
- 📊 Production-ready configuration

## Prerequisites

- A VPS server with Docker installed (Digital Ocean Droplet, Hetzner Cloud instance, etc.)
- A domain name pointing to your server's IP address
- Basic knowledge of command line operations

## Quick Start

### 1. Prepare Your Server

Create a droplet/VPS with Docker installed. Most cloud providers offer Docker-ready images.

### 2. Clone the Repository

```bash
git clone https://github.com/amunim/waha-docker-caddy.git
cd waha-docker-caddy
```

### 3. Create Docker Volumes

```bash
sudo docker volume create caddy_data
sudo docker volume create waha_data
```

### 4. Configure DNS

Set up your DNS records so your domain points to your server's IP address:
- Create an A record for your subdomain (e.g., `waha.yourdomain.com`) pointing to your server's IP

### 5. Open Required Ports

```bash
sudo ufw allow 80
sudo ufw allow 443
```

### 6. Configure Environment Settings

Edit the environment file:

```bash
vi .env
# or if you prefer nano:
nano .env
```

Update the following variables:
- `DATA_FOLDER`: Set to your current directory path (e.g., `/root/waha-docker-caddy`)
- `DOMAIN_NAME`: Your domain (e.g., `yourdomain.com`)
- `SUBDOMAIN`: Your subdomain (e.g., `waha`)
- `SSL_EMAIL`: Your email for SSL certificate notifications

Example `.env` configuration:
```env
DATA_FOLDER=/root/waha-docker-caddy
DOMAIN_NAME=yourdomain.com
SUBDOMAIN=waha
GENERIC_TIMEZONE=Europe/Berlin
SSL_EMAIL=your-email@yourdomain.com
```

### 7. Configure Caddy (Optional)

Edit the Caddyfile to customize your setup:

```bash
vi caddy_config/Caddyfile
```

#### With Basic Authentication (Recommended)

```caddyfile
waha.yourdomain.com {
    basicauth {
        admin $2a$14$VtN0UCkqkBsKS3A.2OQhCeKZ
    }
    reverse_proxy waha:3000 {
        flush_interval -1
    }
}
```

#### Without Basic Authentication

```caddyfile
waha.yourdomain.com {
    reverse_proxy waha:3000 {
        flush_interval -1
    }
}
```

#### Generate Hashed Password

To generate a hashed password for basic authentication:

```bash
docker run --rm caddy caddy hash-password --plaintext "YourSecurePassword"
```

Replace the hash in the Caddyfile with the generated hash.

### 8. Start the Services

```bash
sudo docker compose up -d
```

## Verification

1. Check if containers are running:
   ```bash
   sudo docker compose ps
   ```

2. Check logs if needed:
   ```bash
   sudo docker compose logs -f
   ```

3. Access your WAHA instance at `https://your-subdomain.your-domain.com`

## Project Structure

```
waha-docker-caddy/
├── docker-compose.yml          # Docker services configuration
├── .env                        # Environment variables
├── caddy_config/
│   └── Caddyfile              # Caddy reverse proxy configuration
└── README.md                   # This file
```

## Services

### WAHA
- **Image**: `devlikeapro/waha:latest`
- **Port**: 3000 (internal)
- **Data**: Stored in `waha_data` volume

### Caddy
- **Image**: `caddy:latest`
- **Ports**: 80, 443
- **Features**: Automatic HTTPS, reverse proxy
- **Data**: Stored in `caddy_data` volume

## Cloud Provider Specific Notes

### Digital Ocean

1. Create a Droplet with Docker pre-installed
2. Follow the standard setup above
3. Ensure your domain's DNS is pointed to the Droplet's IP

### Hetzner Cloud

1. Create a Cloud Server with Docker
2. Follow the standard setup above
3. Configure your domain's DNS to point to the server's IP

### Other Providers

This setup works with any cloud provider that supports Docker. Just ensure:
- Docker is installed
- Ports 80 and 443 are accessible
- Your domain points to the server

## Troubleshooting

### SSL Certificate Issues
- Ensure your domain correctly points to your server
- Check that ports 80 and 443 are open
- Verify your email is correct in the `.env` file

### Container Not Starting
```bash
sudo docker compose logs [service-name]
```

### Permission Issues
```bash
sudo chown -R $USER:$USER /path/to/waha-docker-caddy
```

## Security Considerations

1. **Enable Basic Authentication**: Protect your WAHA instance with username/password
2. **Use Strong Passwords**: Generate secure passwords for basic auth
3. **Keep Updated**: Regularly update your containers
4. **Firewall**: Only open necessary ports (80, 443)

## Maintenance

### Update Containers
```bash
sudo docker compose pull
sudo docker compose up -d
```

### Backup Data
```bash
sudo docker run --rm -v waha_data:/data -v $(pwd):/backup alpine tar czf /backup/waha_backup.tar.gz /data
```

### View Logs
```bash
sudo docker compose logs -f waha
sudo docker compose logs -f caddy
```

## Additional Resources

- [WAHA Documentation](https://waha.devlike.pro/)
- [Caddy Documentation](https://caddyserver.com/docs/)
- [Detailed N8N Setup Guide](https://docs.n8n.io/hosting/installation/server-setups/digital-ocean) (Similar setup inspiration)

## Support

If you encounter issues:
1. Check the logs using `sudo docker compose logs`
2. Verify your DNS configuration
3. Ensure all environment variables are correctly set
4. Check that required ports are open

---

**Note**: This setup is inspired by similar production deployments and follows best practices for containerized applications with automatic SSL certificate management i.e. https://docs.n8n.io/hosting/installation/server-setups/digital-ocean
