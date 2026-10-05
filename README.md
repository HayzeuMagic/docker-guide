# docker-guide

PART A

# A1
sudo apt update

sudo apt install -y ca-certificates curl

sudo install -m 0755 -d /etc/apt/keyrings

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc

sudo chmod a+r /etc/apt/keyrings/docker.asc

head -1 /etc/apt/keyrings/docker.asc # must print BEGIN PGP PUBLIC KEY BLOCK

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \

https://download.docker.com/linux/ubuntu \

$(. /etc/os-release && echo "$UBUNTU_CODENAME") stable" \

| sudo tee /etc/apt/sources.list.d/docker.list

# A2
sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io \

docker-buildx-plugin docker-compose-plugin

# A3
sudo usermod -aG docker $USER

sudo -iu $USER # fresh login shell with new group (or log out/in)

docker run hello-world

PART D

# D1
mkdir -p ~/zabbix && cd ~/zabbix

cat > docker-compose.yml <<'EOF'
services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbix
    volumes:
      - zbx-db:/var/lib/postgresql/data

  zabbix-server:
    image: zabbix/zabbix-server-pgsql:alpine-7.0-latest
    depends_on:
      - postgres
    environment:
      DB_SERVER_HOST: postgres
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbix
    ports:
      - "10051:10051"

  zabbix-web:
    image: zabbix/zabbix-web-nginx-pgsql:alpine-7.0-latest
    depends_on:
      - zabbix-server
    environment:
      ZBX_SERVER_HOST: zabbix-server
      DB_SERVER_HOST: postgres
      POSTGRES_USER: zabbix
      POSTGRES_PASSWORD: zabbix
      POSTGRES_DB: zabbix
      PHP_TZ: Asia/Manila
    ports:
      - "8080:8080"

  zabbix-agent:
    image: zabbix/zabbix-agent2:alpine-7.0-latest
    environment:
      ZBX_HOSTNAME: "Zabbix server"
      ZBX_SERVER_HOST: zabbix-server

volumes:
  zbx-db:
EOF

docker compose config --quiet && echo "OK: docker-compose.yml is valid"

# D2
docker compose up -d
docker compose ps # all 4 Up, web (healthy)
docker compose logs zabbix-server | grep -iE "schema|started|error|cannot" | tail -20

# D3
Check
http://127.0.0.1:8080 
user: Admin
pass: zabbix
