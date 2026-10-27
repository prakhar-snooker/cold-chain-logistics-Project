# Ingesting data

## 1. Get the dataset

- Download the dataset from the Kaggle link in `data/source/data.txt`
- Save the CSV as `data/raw/dynamic_supply_chain_logistics_dataset.csv`

## 2. Spin up the legacy MSSQL Server (Docker)

- Image: `mcr.microsoft.com/mssql/server:2022-latest`
- Can run locally (Docker Desktop) or on an EC2 instance with Docker installed
- Replace `<YourStrongPassword>` with your own password (min. 8 chars; uppercase, lowercase, numbers, symbols). Keep the real password only in `.env`.

```
# Windows PowerShell (single line, with a volume so data survives if the container is removed)
docker run -v mssql_data:/var/opt/mssql -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrongPassword>" -p 1433:1433 --name legacy-mssql -d mcr.microsoft.com/mssql/server:2022-latest

# or multi-line in Windows Command Prompt
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrongPassword>" ^
   -p 1433:1433 --name legacy-mssql ^
   -d mcr.microsoft.com/mssql/server:2022-latest

# or multi-line on Linux / macOS
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=<YourStrongPassword>" \
   -p 1433:1433 --name legacy-mssql \
   -d mcr.microsoft.com/mssql/server:2022-latest

# or multi-line with volume (e.g. inside EC2)
docker run -v mssql_data:/var/opt/mssql \
  -e "ACCEPT_EULA=Y" \
  -e "MSSQL_SA_PASSWORD=<YourStrongPassword>" \
  -p 1433:1433 \
  --name legacy-mssql \
  -d mcr.microsoft.com/mssql/server:2022-latest
```

- Check it is running: `docker ps` (status should be `Up`)
- Start it again later (e.g. after a reboot): `docker start legacy-mssql`
- Test the login:

```
docker exec -it legacy-mssql /opt/mssql-tools18/bin/sqlcmd -S localhost -U sa -P "<YourStrongPassword>" -C -Q "SELECT @@VERSION"
```

## 3. Configure the connection

- Create a `.env` file in the project root (it is listed in `.gitignore`, never commit it):

```
SQL_SERVER_HOST=localhost
SQL_SERVER_PORT=1433
SQL_ADMIN_USER=sa
SQL_ADMIN_PASSWORD=<YourStrongPassword>
```

- Requires Microsoft ODBC Driver 18 for SQL Server (64-bit)

## 4. Install the requirements and ingest the data

```
py -3.12 -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
python scripts/ingest_legacy_data.py
```

- Data is loaded into `master.dbo.TBL_SC_FLEET_HIST_RAW`