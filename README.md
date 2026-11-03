# Okta (RBAC) & OpenFGA (Fine-Grained Authorization) Lab

This project demonstrates how to combine Authentication (Okta) with Fine-Grained Authorization (OpenFGA) to protect resources based on user relationships (e.g., "Only the document owner can edit it").

## 🏗️ Architecture
1. **Okta (RBAC):** Authenticates the user and provides an ID Token containing their group memberships (e.g., `Report-Viewers`).
2. **OpenFGA (ReBAC):** Evaluates relationships (e.g., `user:testuser1` is the `owner` of `document:123`) to grant or deny access at the resource level.
3. **Docker + SQLite:** Used to run the OpenFGA server locally with persistent storage.

## 🚀 Prerequisites
- Windows Subsystem for Linux (WSL) with Ubuntu.
- Docker installed in WSL.
- Okta Developer Account.

## 🛠️ Quick Start Commands

### 1. Start Docker and OpenFGA Server
```bash
sudo service docker start

# Initialize the SQLite database (Run only once)
sudo docker run --rm -v /tmp:/tmp openfga/openfga migrate --datastore-engine sqlite --datastore-uri 'file:/tmp/openfga.db'

# Start the server (Keep this terminal open)
sudo docker run -p 8080:8080 -v /tmp:/tmp openfga/openfga run --datastore-engine sqlite --datastore-uri 'file:/tmp/openfga.db'

### 2. Open a new terminal tab and install the CLI
bash
curl -fL -o fga.tar.gz "https://github.com/openfga/cli/releases/download/v0.7.20/fga_0.7.20_linux_amd64.tar.gz"
tar -xzf fga.tar.gz
sudo mv fga /usr/local/bin/
### 3. Create Store and Model
bash
fga store create --name "demo-store"
# Copy the <STORE_ID> from the output

fga model write --store-id <STORE_ID> --file model.fga
### 4. Add Data and Test
bash
# Give testuser1 ownership of document 123
fga tuple write --store-id <STORE_ID> user:testuser1 owner document:123

# Test: Owner can edit (Returns true)
fga query check --store-id <STORE_ID> user:testuser1 can_edit document:123

# Test: Stranger cannot edit (Returns false)
fga query check --store-id <STORE_ID> user:bob can_edit document:123
