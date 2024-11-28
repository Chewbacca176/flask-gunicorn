# flask-gunicorn
## 1. Opprett en Hyper-V virtuell maskin med Ubuntu
### 1.	Installer Hyper-V:
```bash
Åpne Kontrollpanel, velg «programmer», «programmer og funksjoner», «slå windows funksjoner på eller av» og aktiver Hyper-V.
Start PC-en på nytt hvis nødvendig. 
```
### 2.	Last ned Ubuntu ISO:
```bash
Gå til https://releases.ubuntu.com/noble/ og last ned ISO-filen.
Last ned desktop versjonen
```
### 3.	Konfigurer Hyper-V-maskinen:
```bash
Åpne Hyper-V Manager.
Klikk på «quick create», “local installation source”, “change installation source” og velg ubuntu iso filen fra nedlastninger 
Sørg for at
"This virutal machine will run windows
(Enables secure windows boot)"
ikke er på 
```
```bash
Start maskinen, installer Ubuntu, velg norsk keyboard og behold de automatisk valgte mulighetene 
Når du vm-en booter og du kommer inn så må du sjekke om du har på norsk keyboard, hvis du ikke har det så må du bytte det ved å gå til settings som du kan finne ved å trykke på iconet nede i venstre hjørne.
Når du er i settings så må du trykke på keyboard -> input sources -> add input source og legge til norsk 
For å bytte keyboard layout så må du trykke på en av knappene i høyre topp hjørne 
```
## 2. Konfigurer Ubuntu
### 1.	Oppdater systemet: Åpne terminalen og kjør følgende kommandoer:
```bash
sudo apt update && sudo apt upgrade 
```
### 2.	Installer nødvendige pakker:
```bash
sudo apt install python3 python3-pip python3-venv 
```
## 3. Sett opp Flask-applikasjonen
### 1.	Opprett et prosjekt: Lag en mappe for Flask-prosjektet ditt:
```bash
mkdir flask_app && cd flask_app
```
### 2.	Lag et virtuelt miljø:
```bash
python3 -m venv venv
source venv/bin/activate
```
### 3.	Installer Flask og Gunicorn:
```bash
pip install flask gunicorn
```
### 4.	Opprett Flask-applikasjonen: Lag en fil kalt app.py:
```bash
Bruk touch for å lage en fil og bruk nano for å skrive i filen:
```
```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Gunicorn with Flask!"

if __name__ == "__main__":
    app.run()
```
## 4. Test Flask med Gunicorn
### 1.	Start Flask-applikasjonen med Gunicorn:
```bash
gunicorn -w 3 -b 0.0.0.0:8000 app:app
-w 3: Antall arbeidere (juster basert på behov).
-b 0.0.0.0:8000: Lytter på alle IP-er på port 8000.
```
### 2.	Test applikasjonen:
```bash
Finn IP-adressen til den virtuelle maskinen ved å kjøre:
ipa eller ip a
Åpne en nettleser på Windows og gå til "VM_IP":8000.
```

## 5. Sett opp Gunicorn som en systemtjeneste
### 1.	Opprett en systemd-tjenestefil: Lag en fil for tjenesten:
```bash
sudo nano /etc/systemd/system/flask_app.service
Legg til følgende innhold:
```
```bash
[Unit]
Description=Gunicorn instance to serve Flask application
After=network.target

[Service]
User=ubuntu
Group=www-data
WorkingDirectory=/path/to/flask_app
Environment="PATH=/path/to/flask_app/venv/bin"
ExecStart=/path/to/flask_app/venv/bin/gunicorn -w 3 -b 0.0.0.0:8000 app:app

[Install]
WantedBy=multi-user.target
```
### 2.	Start og aktiver tjenesten:
```bash
sudo systemctl start flask_app
sudo systemctl enable flask_app
```
### 3.	Sjekk tjenestestatus:
```bash
sudo systemctl status flask_app
```
```bash
Nå funker flask-gunicorn backenden, resten er ikke nødvendig
```
## 6.  Sett opp NGINX som en omvendt proxy (valgfritt + jeg fikk det ikke helt til å funke)
### 1.	Installer NGINX:
```bash
sudo apt install nginx 
```
### 2.	Konfigurer NGINX: Opprett en ny konfigurasjonsfil:
```bash
sudo nano /etc/nginx/sites-available/flask_project
Legg til følgende innhold:
```
```bash
server {
    listen 80;
    server_name your_domain_or_ip;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
### 3.	Aktiver konfigurasjonen: Opprett en symbolsk lenke:
```bash
sudo ln -s /etc/nginx/sites-available/flask_project /etc/nginx/sites-enabled
```
### 4.	Test NGINX-konfigurasjonen:
```bash
sudo nginx -t
```
### 5.	Start NGINX:
```bash
sudo systemctl restart nginx
```
## 7.   Optimalisering og testing
```bash
Test ytelsen med verktøy som Apache Benchmark (ab):
ab -n 100 -c 10 http://<server-ip>/
Juster antall Gunicorn-arbeidere (-w) basert på serverens ressurser:
Antall arbeidere = 2 x CPU-kjerner + 1.
```
