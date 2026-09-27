
##### Bring up a backend server:
```bash
python -m uvicorn app.main:app --reload
```

##### Bring up a frontend server:
```bash
npm run dev -- --host 127.0.0.1
```

##### Run PostgreSQL container:
```bash
sudo docker run -d \
  --name avatair-postgresql \
  -e POSTGRES_USER=admin \
  -e POSTGRES_PASSWORD=supersecret \
  -e POSTGRES_DB=avatair \
  -p 5432:5432 \
  -v postgres_data:/var/lib/postgresql/data \
  postgres:16-alpine
```