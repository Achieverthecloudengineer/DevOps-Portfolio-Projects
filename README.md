This comprehensive guide is designed to be published as a high-level technical blog post or portfolio case study. It demonstrates your ability to navigate the full lifecycle of a cloud-hosted application on **AWS EC2**, from writing performant code to implementing enterprise-grade process management and data recovery.

---

# Engineering a Resilient Asset Management Appliance on AWS EC2

## Executive Summary

This project showcases the deployment of **InventoryOS**, a professional-grade internal tool for infrastructure tracking. Moving beyond local development, this guide covers the architectural transition to **AWS**, implementing **PM2** for process resiliency, and **Cron** for automated disaster recovery.

## Tools and Technologies needed:

- Aws EC2
- Pm2



# 📦 InventoryOS: Resilient Internal Asset Tracking

A professional-grade, full-stack inventory management system engineered for high availability and minimal operational overhead. This project demonstrates a **"Simplicity-First"** architecture, opting for a unified Node.js appliance over complex build-chains to optimize deployment speed and maintainability.

---

## 🏗️ Architectural Overview

This system follows a **Monolithic Appliance** pattern, where the Express.js backend serves as both the RESTful API provider and the static file host for the React frontend.

* **Runtime:** Node.js (V8)
* **Backend:** Express.js REST API
* **Frontend:** React 18 (Client-side rendering via CDN)
* **Styling:** Custom CSS3 (Obsidian/DevOps Dark Mode)
* **Process Management:** Linux Systemd (Daemonization)

---

## 🚀 Key Features

* **Full CRUD Lifecycle:** Create, Read, Update, and Delete assets via a unified dashboard.
* **Real-time Client-Side Filtering:** High-performance search using React `useMemo` hooks to minimize server-side CPU cycles.
* **DevOps Aesthetics:** Responsive, dark-mode UI utilizing **Lucide-React** icons for a professional command-center feel.
* **Service Persistence:** Configured as a background daemon to survive system reboots and process crashes.

---

## 🛠️ Step-by-Step Implementation

### 1. Backend Engineering (`app.js`)

The server-side logic focuses on predictable routing and lightweight data handling.

`

---


## 1. Provisioning the Cloud Environment (AWS EC2)

Before deploying code, Ensure your configuration matches mine as used on AWS.

### Step 1: Instance Launch

* **AMI:** Amazon Linux 2023 or Ubuntu 22.04 LTS.
* **Instance Type:** t3.micro (Free Tier eligible).
* **Security Group Rules:** * Port 22 (SSH): Restricted to your IP.
* Port 7000 (Custom TCP): Open for the application.



### Step 2: Environment Preparation

Connect via SSH and install the Node.js runtime from below website, ensure to select linux as shown below

https://nodejs.org/en/download


```bash
sudo dnf update -y

```

![alt text](image.png)
---

## 2. Backend Engineering: The Express.js API (`app.js`)

We utilize a RESTful architecture. The server is designed as a unified appliance, serving both the API data and the static frontend assets.

- Create a file called `app.js` and copy below code:

```javascript
const express = require('express');
const app = express();
const PORT = 3000;

app.use(express.json());
app.use(express.static('public')); // Serve frontend from 'public' folder by locating index.html

// This simple app will use In-memory data in our browser to store for high-speed responsiveness
let inventory = [
    { id: 1, name: "Cisco Catalyst 9300", qty: 2, category: "Networking" },
    { id: 2, name: "PowerEdge R750", qty: 5, category: "Servers" }
];

app.get('/api/inventory', (req, res) => res.json(inventory));

app.post('/api/inventory', (req, res) => {
    const newItem = { id: Date.now(), ...req.body };
    inventory.push(newItem);
    res.status(201).json(newItem);
});

app.put('/api/inventory/:id', (req, res) => {
    const { id } = req.params;
    inventory = inventory.map(item => item.id == id ? { ...item, ...req.body } : item);
    res.json({ message: "Update Successful" });
});

app.delete('/api/inventory/:id', (req, res) => {
    inventory = inventory.filter(item => item.id != req.params.id);
    res.status(204).send();
});

app.listen(PORT, '0.0.0.0', () => console.log(`InventoryOS live on port ${PORT}`));

```

---

## 3. Frontend Engineering: The React SPA

This section we will create the UI. The UI is a **Single Page Application (SPA)** optimized for DevOps environments. It features a right-aligned compact search bar, Lucide icons, and a client-side backup engine.

- Create a folder and a file - `public/index.html`) and copy below code inside it.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>InventoryOS | Pro</title>
    <script src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap');
        :root { --bg: #0d1117; --card: #161b22; --border: #30363d; --text: #c9d1d9; --primary: #58a6ff; --success: #238636; --danger: #f85149; }
        body { background: var(--bg); color: var(--text); font-family: 'Inter', sans-serif; margin: 0; padding: 2rem; }
        .container { max-width: 850px; margin: auto; }
        .header { display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--border); padding-bottom: 1rem; margin-bottom: 2rem; }
        .brand { display: flex; align-items: center; gap: 8px; font-weight: 600; font-size: 1.1rem; }
        .header-actions { display: flex; align-items: center; gap: 12px; }
        .search-input { background: #010409; border: 1px solid var(--border); color: white; padding: 6px 10px 6px 30px; border-radius: 6px; width: 140px; font-size: 0.8rem; transition: 0.3s; }
        .search-input:focus { width: 190px; border-color: var(--primary); outline: none; }
        .card { background: var(--card); border: 1px solid var(--border); border-radius: 8px; padding: 1.25rem; margin-bottom: 1.5rem; }
        .input-group { display: flex; gap: 10px; }
        input.field { background: #010409; border: 1px solid var(--border); color: white; padding: 8px; border-radius: 6px; flex: 1; font-size: 0.85rem; }
        button { cursor: pointer; border-radius: 6px; border: none; font-weight: 600; display: flex; align-items: center; gap: 5px; padding: 8px 14px; font-size: 0.85rem; }
        .btn-add { background: var(--success); color: white; }
        .btn-export { background: transparent; color: var(--text); border: 1px solid var(--border); }
        table { width: 100%; border-collapse: collapse; }
        th { text-align: left; color: #8b949e; font-size: 0.7rem; text-transform: uppercase; padding: 10px; border-bottom: 1px solid var(--border); }
        td { padding: 12px 10px; border-bottom: 1px solid var(--border); font-size: 0.9rem; }
    </style>
</head>
<body>
    <div id="root"></div>
    <script type="text/babel">
        const { useState, useEffect, useMemo } = React;
        const Icon = ({ name, size = 16 }) => {
            useEffect(() => { lucide.createIcons(); }, [name]);
            return <i data-lucide={name} style={{ width: size, height: size }}></i>;
        };

        function App() {
            const [items, setItems] = useState([]);
            const [search, setSearch] = useState('');
            const [form, setForm] = useState({ name: '', qty: '', category: '' });
            const [editingId, setEditingId] = useState(null);

            useEffect(() => { fetch('/api/inventory').then(res => res.json()).then(setItems); }, []);

            const filtered = useMemo(() => items.filter(i => i.name.toLowerCase().includes(search.toLowerCase())), [items, search]);

            const exportData = () => {
                const blob = new Blob([JSON.stringify(items, null, 2)], { type: 'application/json' });
                const url = URL.createObjectURL(blob);
                const a = document.createElement('a');
                a.href = url; a.download = 'inventory_backup.json'; a.click();
            };

            const handleSubmit = async (e) => {
                e.preventDefault();
                await fetch(editingId ? `/api/inventory/${editingId}` : '/api/inventory', {
                    method: editingId ? 'PUT' : 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(form)
                });
                setForm({ name: '', qty: '', category: '' }); setEditingId(null);
                fetch('/api/inventory').then(res => res.json()).then(setItems);
            };

            return (
                <div className="container">
                    <header className="header">
                        <div className="brand"><Icon name="server" /> InventoryOS</div>
                        <div className="header-actions">
                            <button className="btn-export" onClick={exportData}><Icon name="download" size={14}/>Backup</button>
                            <div style={{position:'relative'}}>
                                <span style={{position:'absolute', left:'10px', top:'8px', color:'#8b949e'}}><Icon name="search" size={14}/></span>
                                <input className="search-input" placeholder="Filter..." onChange={e => setSearch(e.target.value)} />
                            </div>
                        </div>
                    </header>
                    <div className="card">
                        <form className="input-group" onSubmit={handleSubmit}>
                            <input className="field" placeholder="Item" value={form.name} onChange={e=>setForm({...form, name:e.target.value})} required />
                            <input className="field" placeholder="Category" value={form.category} onChange={e=>setForm({...form, category:e.target.value})} required />
                            <input className="field" type="number" placeholder="Qty" style={{maxWidth:'60px'}} value={form.qty} onChange={e=>setForm({...form, qty:e.target.value})} required />
                            <button className="btn-add"><Icon name="plus"/> {editingId ? 'Save' : 'Add'}</button>
                        </form>
                    </div>
                    <div className="card">
                        <table>
                            <thead><tr><th>Asset</th><th>Category</th><th>Qty</th><th style={{textAlign:'right'}}>Actions</th></tr></thead>
                            <tbody>
                                {filtered.map(i => (
                                    <tr key={i.id}>
                                        <td>{i.name}</td>
                                        <td><span style={{color:'var(--primary)', fontSize:'0.75rem'}}>{i.category}</span></td>
                                        <td>{i.qty}</td>
                                        <td style={{textAlign:'right'}}>
                                            <button style={{display:'inline', background:'none', color:'#8b949e'}} onClick={() => {setEditingId(i.id); setForm(i);}}><Icon name="edit" size={14}/></button>
                                            <button style={{display:'inline', background:'none', color:'var(--danger)'}} onClick={async () => { await fetch(`/api/inventory/${i.id}`, {method:'DELETE'}); window.location.reload(); }}><Icon name="trash" size={14}/></button>
                                        </td>
                                    </tr>
                                ))}
                            </tbody>
                        </table>
                    </div>
                </div>
            );
        }
        ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    </script>
</body>
</html>

```

##  Deploying your code and Testing it

































---

## 4. Production Orchestration: PM2 & Automation

To ensure the app survives the "Real World," we use **PM2**. It provides zero-downtime reloads and automatic self-healing.

### The Deployment Script (`deploy.sh`)

This script automates the setup on your EC2 instance.

```bash
#!/bin/bash
# Install PM2 Globally
sudo npm install -g pm2

# Install local dependencies
npm install

# Start the application as a background daemon
pm2 start app.js --name "inventory-os"

# Configure PM2 to start on system boot
pm2 save
pm2 startup | tail -n 1 | bash

echo "✅ Deployment Successful"

```

---

## 5. Reliability Engineering: Automated Backups

A Senior DevOps Engineer never trusts "Local" state. We implement a **Cron Job** to take nightly snapshots of the API data.

### The Backup Script (`backup.sh`)

```bash
#!/bin/bash
BACKUP_DIR="/home/ec2-user/backups"
mkdir -p $BACKUP_DIR
# Pull data from the local API
curl -s http://localhost:3000/api/inventory > "$BACKUP_DIR/inv_$(date +%F).json"
# Retention Policy: Delete backups older than 7 days
find $BACKUP_DIR -type f -mtime +7 -delete

```

### Scheduling the Backup

Run `crontab -e` and add:
`0 0 * * * /bin/bash /home/ec2-user/backup.sh`

---

## 6. Verification: Post-Deployment Steps

After deployment, verify the stack using these commands:

1. **Check Process:** `pm2 status`
2. **Monitor Logs:** `pm2 logs inventory-os`
3. **Monitor Health:** `pm2 monit` (This provides a real-time terminal dashboard of CPU/Memory usage).

---

### Conclusion

By deploying on **AWS EC2** with **PM2** and **Automated Backups**, this project demonstrates a complete understanding of the **Reliability Pillar** of the AWS Well-Architected Framework.

**Would you like me to help you configure an Nginx Reverse Proxy with SSL so you can access this app via a secure domain name?**
