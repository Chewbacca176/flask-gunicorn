# flask-gunicorn
```bash
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello, Gunicorn with Flask!"

if __name__ == "__main__":
    app.run()
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
