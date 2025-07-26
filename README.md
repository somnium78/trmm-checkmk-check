# TRMM CheckMK Local Check

A CheckMK local check script for monitoring Tactical RMM (TRMM) installations. This script provides comprehensive monitoring of your TRMM server including disk usage, memory usage, certificate expiration, service status, and connectivity checks.

## Features

- **System Resources**: Monitor disk and memory usage with configurable thresholds
- **Certificate Monitoring**: Track SSL certificate expiration dates
- **Service Status**: Monitor all TRMM services (mesh, daphne, celery, redis, nats)
- **Queue Health**: Monitor Celery queue length and health status
- **Connectivity**: Check Redis, NATS, and MeshCentral connectivity
- **Agent Statistics**: Track managed agents, clients, and sites

## Prerequisites

### Required Packages

Install the following packages on your TRMM server:

- apt install curl jq
  oder
- dnf install curl jq

### TRMM Configuration

1. **Enable Monitoring Token** in your TRMM installation:

   Add the following line to `/rmm/api/tacticalrmm/tacticalrmm/local_settings.py`:

       MON_TOKEN = "your_secure_monitoring_token_here"

2. **Restart TRMM Service**:

       sudo systemctl restart rmm.service

3. **Configure Nginx** for local API access:

   Add the following server block to `/etc/nginx/sites-enabled/rmm.conf`:

       server {
           listen 127.0.0.1:8080;
           server_name api.yourdomain.com;

           # Only allow local access
           allow 127.0.0.1;
           deny all;

           # TRMM API configuration
           location / {
               uwsgi_pass  tacticalrmm;
               include     /etc/nginx/uwsgi_params;
               uwsgi_read_timeout 300s;
               uwsgi_ignore_client_abort on;
           }
       }

4. **Reload Nginx**:

       sudo nginx -t
       sudo systemctl reload nginx

## Installation

1. **Download the script**:

       wget https://raw.githubusercontent.com/somnium78/trmm-checkmk-check/main/trmm_status

2. **Configure the script**:
   Edit the configuration section at the top of the script:

       nano trmm_status

   Modify these variables:
   - `TOKEN`: Your TRMM monitoring token
   - `API_HOSTNAME`: Your TRMM API hostname (e.g., api.yourdomain.com)
   - Adjust thresholds as needed

3. **Install the script**:

       sudo cp trmm_status /usr/lib/check_mk_agent/local/
       sudo chmod +x /usr/lib/check_mk_agent/local/trmm_status
       sudo chown root:root /usr/lib/check_mk_agent/local/trmm_status

4. **Test the script**:

       /usr/lib/check_mk_agent/local/trmm_status

## CheckMK Integration

1. **Service Discovery**: Run service discovery on your CheckMK server:

       cmk -I your_trmm_hostname
       cmk -R your_trmm_hostname

2. **Services**: The following services will be created:
   - `TRMM_Agents`: Agent count and statistics
   - `TRMM_Certificate`: SSL certificate expiration monitoring
   - `TRMM_Connectivity`: Redis, NATS, and MeshCentral connectivity
   - `TRMM_Disk`: Disk usage monitoring
   - `TRMM_Memory`: Memory usage monitoring
   - `TRMM_Queue`: Celery queue monitoring
   - `TRMM_Services`: TRMM service status monitoring

## Configuration

### Thresholds

You can adjust the monitoring thresholds by modifying these variables in the script:

    DISK_WARN_THRESHOLD=80      # Disk usage warning at 80%
    DISK_CRIT_THRESHOLD=90      # Disk usage critical at 90%
    MEMORY_WARN_THRESHOLD=80    # Memory usage warning at 80%
    MEMORY_CRIT_THRESHOLD=90    # Memory usage critical at 90%
    CERT_WARN_DAYS=30          # Certificate warning at 30 days
    CERT_CRIT_DAYS=7           # Certificate critical at 7 days
    QUEUE_WARN_LENGTH=50       # Queue warning at 50 items
    QUEUE_CRIT_LENGTH=100      # Queue critical at 100 items

## Troubleshooting

### Common Issues

1. **"curl is not installed"**: Install curl package
2. **"jq is not installed"**: Install jq package
3. **"TRMM API not responding"**: Check nginx configuration and monitoring token
4. **Empty response**: Verify the monitoring token in local_settings.py

### Testing API Access

Test the API manually:

    curl -s -H "Host: api.yourdomain.com" \
         -H "X-Mon-Token: your_token_here" \
         "http://127.0.0.1:8080/core/v2/status/"

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the GNU General Public License v3.0 - see the [LICENSE](LICENSE) file for details.

### Why GPL v3?

This project uses GPL v3 to ensure that any improvements or modifications to this monitoring script remain open source and benefit the entire community. This aligns with the open source philosophy of both Tactical RMM and CheckMK.

## Disclaimer

**DISCLAIMER OF WARRANTY AND LIMITATION OF LIABILITY**

This software is provided "as is" without warranty of any kind, express or implied, including but not limited to the warranties of merchantability, fitness for a particular purpose, and noninfringement. In no event shall the authors or copyright holders be liable for any claim, damages, or other liability, whether in an action of contract, tort, or otherwise, arising from, out of, or in connection with the software or the use or other dealings in the software.

Use this software at your own risk. The authors are not responsible for any damage or data loss that may occur from using this software.

## Support

For issues and questions:
- Create an issue on [GitHub](https://github.com/somnium78/trmm-checkmk-check/issues)
- Check the TRMM documentation: https://docs.tacticalrmm.com/
- CheckMK documentation: https://docs.checkmk.com/

---

**Note**: This is an unofficial monitoring script and is not affiliated with or endorsed by Tactical RMM or CheckMK.
