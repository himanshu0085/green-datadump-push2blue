# ✅ Green → Blue MySQL Cutover – POC Procedure

## 🎯 Objective
To migrate the MySQL database from **Green** environment to **Blue** environment using a MySQL dump and validate successful cutover.

---

## 🟢 GREEN Environment Steps

### ✅ Step 1: Create dump directory & take MySQL backup

```bash
mkdir cutover
cd cutover
ls
docker exec -i db mysqldump --single-transaction \
  -u root -p'password' buildpiper > BP-dump-$(date +%d%b%Y).sql
ls
# BP-dump-29Oct2025.sql
```

Backup created at:

```
/home/ubuntu/green_dump/BP-dump-29Oct2025.sql
```

---

### ✅ Step 2: Transfer dump from Green → Blue

```bash
scp BP-dump-29Oct2025.sql ubuntu@10.51.7.233:/home/ubuntu/green_dump/
```

---

## 🔵 BLUE Environment Steps

### ✅ Step 3: Move dump to working directory

```bash
sudo su buildpiper
mkdir -p ~/cutover
cp /home/ubuntu/green_dump/BP-dump-29Oct2025.sql ~/cutover/
cd ~/cutover
ls
# BP-dump-29Oct2025.sql
```

---

### ✅ Step 4: Stop dependent services (safe cutover)

```bash
docker rm -f scheduledapi publicapi
```

---

### ✅ Step 5: Drop & recreate database in Blue env

```bash
docker exec -it db mysql -u root -p'password' -e "DROP DATABASE buildpiper;"
docker exec -it db mysql -u root -p'password' -e "CREATE DATABASE buildpiper;"
```

---

### ✅ Step 6: Import Green dump into Blue DB

```bash
pv BP-dump-29Oct2025.sql | docker exec -i db \
mysql -u root -p'password' buildpiper
```

✅ Successful data import

---

### ✅ Step 7: Restart full application stack

```bash
cd ~/deploy_buildpiper/
docker compose up -d
```

Check logs:

```bash
docker logs -f publicapi
```

---

## ✅ Verification Checklist

| Validation Item | Status |
|----------------|:------:|
| DB refreshed with Green data | ✅ |
| App containers running | ✅ |
| Basic application functionality validated | ✅ |

---

## 🔐 Recommendations & Notes

- Avoid storing passwords in command line history
- Backup Blue DB before dropping (rollback protection)
- Use timestamps in filenames for versioning
- Perform smoke tests after cutover

---

## 📌 Status

**POC Completed Successfully**  
✅ Blue environment running with Green environment data

---
