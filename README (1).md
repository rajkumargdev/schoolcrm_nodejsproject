# SchoolCRM — Class 7 Marks Tracker

A full-stack web application to track student marks across six subjects. Built with **Node.js/Express** backend, **React** frontend, and **PostgreSQL** database. Deployed on AWS EC2.

---

## Project Structure

```
schoolcrm_nodejsproject/
├── backend-node/
│   └── backend-node/
│       ├── src/
│       │   └── index.js        # Main Express server entry point
│       ├── .env.example        # Environment variable template
│       ├── package.json        # Node.js dependencies
│       └── node_modules/       # Installed packages (not committed)
├── frontend/
│   ├── src/                    # React source code
│   ├── public/                 # Static assets
│   ├── build/                  # Production build output (after npm run build)
│   └── package.json
└── db/
    └── migrations/
        ├── 001_create_tables.sql
        ├── 002_create_indexes.sql
        └── 003_seed_data.sql
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Node.js + Express |
| Frontend | React (Create React App) |
| Database | PostgreSQL |
| Auth | JWT (JSON Web Tokens) |
| Password Hashing | bcrypt |
| Hosting | AWS EC2 (Ubuntu) |

---

## What Each Part Does

### Backend (Node.js/Express)
- Runs on port **8080**
- Connects to PostgreSQL using `DATABASE_URL` from `.env`
- Handles REST API routes for login, marks, students, teachers
- Uses **bcrypt** to hash passwords before storing in DB
- Uses **JWT** to issue tokens after login — every protected API request must include this token in the header

### Frontend (React)
- Built with Create React App
- Has a **Teacher panel** and **Student panel**
- API base URL is set in `src/api.js` — must be updated to your EC2 public IP before building
- `npm run build` produces a static `build/` folder served via `serve`

### Database (PostgreSQL)
- Database name: `marks_tracker`
- User: `marks_user`
- Schema and seed data defined in `db/migrations/` folder

---

## Environment Variables

Create a `.env` file inside `backend-node/backend-node/`:

```env
DATABASE_URL=postgresql://marks_user:marks_pass_2024@localhost:5432/marks_tracker
JWT_SECRET=your_random_secret_here
PORT=8080
```

Generate a secure JWT secret:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

> ⚠️ Never commit `.env` to git — it is listed in `.gitignore`

---

## Deployment on AWS EC2 (Ubuntu)

### 1. Clone the Repo
```bash
git clone https://github.com/rajkumargdev/schoolcrm_nodejsproject.git
```

### 2. Install PostgreSQL
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install postgresql postgresql-contrib -y
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### 3. Setup Database
```bash
sudo -u postgres psql
```
```sql
CREATE USER marks_user WITH PASSWORD 'marks_pass_2024';
CREATE DATABASE marks_tracker OWNER marks_user;
GRANT ALL PRIVILEGES ON DATABASE marks_tracker TO marks_user;
\q
```

### 4. Run Migrations
```bash
cd schoolcrm_nodejsproject/db/migrations

psql -U marks_user -d marks_tracker -h localhost -f 001_create_tables.sql
psql -U marks_user -d marks_tracker -h localhost -f 002_create_indexes.sql
psql -U marks_user -d marks_tracker -h localhost -f 003_seed_data.sql
# password: marks_pass_2024
```

### 5. Install Node.js 22
```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v && npm -v
```

### 6. Install Backend Dependencies
```bash
cd ~/schoolcrm_nodejsproject/backend-node/backend-node
npm install
```

### 7. Create .env File
```bash
cp .env.example .env
vi .env
# Fill in your values
```

### 8. Update Frontend API URL
```bash
vi ~/schoolcrm_nodejsproject/frontend/src/api.js
# Replace hardcoded IP with your EC2 public IP
# Example: http://<your-ec2-public-ip>:8080
```

### 9. Build Frontend
```bash
cd ~/schoolcrm_nodejsproject/frontend
npm install
npm run build
```

### 10. Serve Frontend
```bash
sudo npm install -g serve
serve -s build -l 3000
```

### 11. Run Backend (persistent)
Open a second terminal:
```bash
cd ~/schoolcrm_nodejsproject/backend-node/backend-node
nohup node src/index.js > app.log 2>&1 &
tail -f app.log
```

### 12. Open EC2 Security Group Ports
In AWS Console → EC2 → Security Groups → Inbound Rules:

| Port | Purpose |
|------|---------|
| 22   | SSH |
| 8080 | Node.js Backend API |
| 3000 | React Frontend |

---

## Access the App

| What | URL |
|------|-----|
| Frontend | `http://<your-ec2-public-ip>:3000` |
| Backend API | `http://<your-ec2-public-ip>:8080` |

---

## Useful Commands

### Check backend is running
```bash
ps aux | grep node
tail -f ~/schoolcrm_nodejsproject/backend-node/backend-node/app.log
```

### Kill backend
```bash
kill <PID>
```

### Restart backend
```bash
cd ~/schoolcrm_nodejsproject/backend-node/backend-node
nohup node src/index.js > app.log 2>&1 &
```

### Check DB manually
```bash
psql -U marks_user -d marks_tracker -h localhost
# password: marks_pass_2024
\dt   # list tables
\q    # quit
```

---

## Common Mistakes to Avoid

| Mistake | Fix |
|---------|-----|
| JWT_SECRET wrapped in `< >` | Remove angle brackets, paste raw hex string |
| Forgot to update frontend API URL before build | Edit `src/api.js`, re-run `npm run build` |
| Port not open in Security Group | Add inbound rule in AWS Console |
| Backend dies after terminal closes | Use `nohup` as shown in Step 11 |
| `npm install` run in wrong folder | Backend: `backend-node/backend-node/` — Frontend: `frontend/` |

---

## Roadmap

- [x] Rust/Axum backend (v1)
- [x] Node.js/Express backend (v2)
- [ ] Python/FastAPI backend (v3)
- [ ] Dockerize all three backends
- [ ] Deploy on K3s/Kubernetes
