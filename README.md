# Property Manager

Property Manager is a self-hosted application for tracking repairs, expenses, receipts, mileage,
documents, and vendors across your rental properties. Data is stored in a single SQLite file, so no
database server is required.

Each release ships self-contained binaries (no .NET runtime needed) for Windows and Linux, plus a
Docker image.

## Download

Grab the latest release assets from the [Releases](https://github.com/stoxello/PropertyManager-Releases/releases) page:

- `PropertyManager-vX.Y.Z-windows-x64.zip` — Windows x64
- `PropertyManager-vX.Y.Z-linux-x64.tar.gz` — Linux x64
- `SHA256SUMS.txt` — checksums for the archives above

Verify the download against the published checksum:

```bash
# Linux
sha256sum -c SHA256SUMS.txt --ignore-missing

# Windows (PowerShell)
Get-FileHash .\PropertyManager-vX.Y.Z-windows-x64.zip -Algorithm SHA256
```

## Windows install

1. Download `PropertyManager-vX.Y.Z-windows-x64.zip` and extract it to a folder, for example
   `C:\PropertyManager`:

   ```powershell
   Expand-Archive .\PropertyManager-vX.Y.Z-windows-x64.zip -DestinationPath C:\PropertyManager
   ```

2. Run the application:

   ```powershell
   cd C:\PropertyManager
   .\RentalManager.Web.exe
   ```

3. Open <http://localhost:5000> in a browser. The first run creates the database, seeds the roles,
   and redirects you to the **First-Time Setup** page (`/account/setup`) to create your admin
   account.

To listen on a different port or on all network interfaces:

```powershell
$env:ASPNETCORE_URLS = "http://0.0.0.0:8080"
.\RentalManager.Web.exe
```

> The application serves plain HTTP only. Put it behind a reverse proxy (Caddy, nginx, IIS, or a
> similar tool) for HTTPS in production.

## Linux install

1. Download the archive and extract it:

   ```bash
   mkdir -p /opt/property-manager && cd /opt/property-manager
   wget https://github.com/stoxello/PropertyManager-Releases/releases/latest/download/PropertyManager-vX.Y.Z-linux-x64.tar.gz
   tar -xzf PropertyManager-vX.Y.Z-linux-x64.tar.gz
   chmod +x RentalManager.Web
   ```

2. Run the application (must have write access to the folder so the database can be created):

   ```bash
   ASPNETCORE_URLS=http://0.0.0.0:8080 ./RentalManager.Web
   ```

3. Open <http://localhost:8080> (or the server's address on your LAN). First run redirects to
   `/account/setup` to create the admin account.

### Run as a systemd service

Create `/etc/systemd/system/property-manager.service`:

```ini
[Unit]
Description=Stoxello Property Manager
After=network.target

[Service]
WorkingDirectory=/opt/property-manager
ExecStart=/opt/property-manager/RentalManager.Web
Environment=ASPNETCORE_URLS=http://0.0.0.0:8080
Restart=always
RestartSec=3
User=property

[Install]
WantedBy=multi-user.target
```

Run it as a dedicated, non-root user that owns `/opt/property-manager`:

```bash
sudo useradd --system --home /opt/property-manager property
sudo chown -R property:property /opt/property-manager
sudo systemctl enable --now property-manager
```

## Docker

A prebuilt image is published to GitHub Container Registry for each release with `X.Y.Z`, `X.Y`,
and `latest` tags. Run it with persistent volumes for `/app/data`, `/app/receipts`, and
`/app/documents`:

```bash
docker run -d \
  --name property-manager \
  -p 8080:8080 \
  -v property-data:/app/data \
  -v property-receipts:/app/receipts \
  -v property-documents:/app/documents \
  ghcr.io/stoxello/property-management:latest
```

## First-run setup

1. Start the application and open the web interface.
2. On the first run you are redirected to `/account/setup` — create your administrator account.
3. Sign in at `/account/login`.
4. A 30-day trial starts on first run. When the trial ends, purchase a license at the
   [store](https://stoxello.com/store) and activate it as an admin under **License**
   (`/account/license`).

## Data and upgrades

All application data lives in the folder the app runs from:

| Data | Location (default) |
|---|---|
| SQLite database | `rental_manager.db` |
| Uploaded receipts | `receipts/` |
| Uploaded documents | `documents/` |
| Data-protection keys | `dp-keys/` |
| License machine ID | `license-machine-id.txt` (next to the database) |

To upgrade to a new release: stop the app, keep the files above, replace the remaining files with
the new release, and start again. The schema is created automatically on first run — no migrations
are required.

Always keep backups. Admins can download a database-only or full backup (database + all uploaded
files) and restore from a `.db` file under **Admin → Settings** (`/admin/settings`).

## Configuration

Edit `appsettings.json` in the release folder, or override any value with an environment variable.

| Setting | Default | Description |
|---|---|---|
| `ASPNETCORE_URLS` | `http://localhost:5000` | Bind address and port |
| `ConnectionStrings__DefaultConnection` | `Data Source=rental_manager.db` | SQLite connection string |
| `AppSettings__ReceiptStoragePath` | `receipts` | Receipt upload folder |
| `AppSettings__DocumentStoragePath` | `documents` | Document upload folder |
| `DataProtection__KeysPath` | `dp-keys` | Data-protection key folder |
| `Licensing__MachineIdPath` | next to the database | License machine-ID file |
| `Licensing__StoreUrl` | `https://stoxello.com/store` | Where to buy a license |
