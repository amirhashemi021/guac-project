# Apache Guacamole in Docker with NGINX Load Balancing

This project sets up an Apache Guacamole remote desktop gateway environment using Docker Compose. It includes:

- Two Guacamole server containers (for redundancy and load balancing)
- A PostgreSQL database for user authentication and connection settings
- A guacd backend daemon for handling remote desktop protocols (RDP, VNC, SSH)
- An NGINX reverse proxy configured to load-balance traffic across Guacamole servers

---

## Project Structure

.
├── docker-compose.yml
├── initdb.sql
└── nginx
└── nginx.conf


---

##  Features

- Full Guacamole stack deployed with Docker
- NGINX reverse proxy for load balancing and fault tolerance
- PostgreSQL as the authentication backend (JDBC module)
- All containers orchestrated via `docker-compose`
- Easy access via `http://localhost/guacamole`

---

##  Getting Started

### 1. Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- Remote desktop target (e.g., VM with RDP/VNC/SSH enabled)

---

### 2. Clone the Repository


git clone https://github.com/amirhashemi021/guac-project.git
cd guacamole-project

3. Generate the Guacamole Database Schema

Before launching the full stack, you need to generate the SQL schema file that initializes the PostgreSQL database for Guacamole.

Run the following command once to create initdb.sql:

docker run --rm guacamole/guacamole /opt/guacamole/bin/initdb.sh --postgresql > initdb.sql

This command:

    Runs a temporary Guacamole container

    Extracts the PostgreSQL-compatible initialization script from it

    Redirects the output to a local file initdb.sql

    Note: This file is used in the next step to populate the database.

4. Initialize the Database Schema

Now start only the database container and load the schema:

# Start only PostgreSQL
docker compose up -d postgres

# Wait a few seconds for PostgreSQL to be ready

# Import the schema into the database
cat initdb.sql | docker exec -i guacamole-project-postgres-1 psql -U guac_admin -d guacamole_db

5. Start the Full Stack

docker compose up -d

Containers:

    postgres — PostgreSQL database

    guacd — Protocol daemon

    guac-server-1 — First Guacamole instance

    guac-server-2 — Second Guacamole instance

    nginx — Reverse proxy/load balancer

Accessing the Web UI

Open a browser and visit:

    http://localhost:8080/guacamole/ → Direct to guac-server-1

    http://localhost:8081/guacamole/ → Direct to guac-server-2

    http://localhost/guacamole/ → Through NGINX load balancer

Default credentials:

Username: guacadmin
Password: guacadmin

⚙️ NGINX Configuration Explained

Located at nginx/nginx.conf:

upstream guacamole_servers {
    ip_hash;
    server guac-server-1:8080;
    server guac-server-2:8080;
}

    ip_hash: Ensures that the same client IP is always routed to the same backend server (useful for session stickiness).

    Load balances requests between the two Guacamole instances.

    /guacamole/ path is proxied to one of the upstream servers via proxy_pass.

Handles:

    Basic load balancing (Round-robin + IP-based stickiness)

    Redundancy (if one Guacamole server fails, NGINX routes to the other)

Administration

Using any of the three URLs above, once logged in as guacadmin, you can:

    Add new connections (RDP, VNC, SSH)

    Create/delete users

    Assign permissions

All changes are stored in PostgreSQL, meaning both Guacamole servers are always in sync since they share the same database.

Persistent Data

The following volume is used:

    postgres_data: Stores PostgreSQL data between restarts

If you ever want a clean slate:

docker compose down -v

Security Recommendations

    Change the default guacadmin credentials immediately.

    Use SSL/TLS (configure NGINX with a certificate).

    Use secure passwords for DB and connections.

    Restrict exposed ports in production (e.g., 8080/8081 may not be needed if using NGINX only).

Extending the Stack

    Add more Guacamole servers in docker-compose.yml and nginx.conf

    Configure TLS termination in NGINX

    Deploy behind a firewall or in a secured network segment

    Use Docker secrets for storing sensitive credentials (optional)

Cleanup

To remove all containers and volumes:

docker compose down -v

License

This project is for educational and internal use only. Apache Guacamole is licensed under the Apache License v2.0.

Questions?
If you run into issues or need improvements (e.g., HTTPS setup, LDAP integration, multi-user permissions), feel free to open an issue or ask.


---

