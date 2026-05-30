# Warehouse — Express Backend

Selbst-gehostetes Backend für die Projekte [data-management](https://github.com/cmariej/data-management) und [clara-berning](https://github.com/cmariej/clara-berning). Läuft als Docker-Container hinter nginx auf dem Heimserver.

## Architektur

- **Express** API auf Port 3000 (nur intern, nginx ist der einzige Eintrittspunkt)
- **nginx** reverse proxy mit SSL (`warehouse.jung-privat.de`)
- Alle persistenten Daten liegen auf dem Host und werden per Volume in den Container gemountet

## Projektstruktur

```
warehouse/
  server.js               ← Einstiegspunkt
  storage.js              ← Dateizugriff (liest/schreibt JSON aus data/)
  routes/
    auth.routes.js        ← POST /api/auth/login
    project.routes.js     ← GET/PUT /api/projects/...
    media.routes.js       ← POST/DELETE /api/media
  middleware/
    auth.js               ← JWT-Middleware
  data/                   ← JSON-Dateien (persistent, nicht im Git)
  media/                  ← Bilder (persistent, nicht im Git)
  users/
    users.json            ← Benutzer mit bcrypt-Passwörtern
  Dockerfile
  docker-compose.yml
  .env                    ← Nicht einchecken
```

## API-Endpunkte

| Methode | Pfad | Auth | Beschreibung |
|---------|------|------|--------------|
| POST | `/api/auth/login` | — | Login, gibt JWT zurück |
| GET | `/api/projects` | — | Projektliste laden |
| GET | `/api/projects/:project/:file` | — | JSON-Datei laden |
| GET | `/api/projects/:project/:file/schema` | — | Schema laden |
| PUT | `/api/projects/:project/:file` | JWT | JSON-Datei speichern |
| POST | `/api/media` | JWT | Bild hochladen |
| DELETE | `/api/media` | JWT | Bild löschen |
| GET | `/media/...` | — | Statische Bilder |

## Lokale Entwicklung

```bash
docker compose up -d --build
```

Logs:
```bash
docker compose logs -f
```

Erwartete Ausgabe: `Server läuft auf 3000`

## Deployment (Heimserver)

### Ersteinrichtung

```bash
git clone https://github.com/cmariej/warehouse.git
cd warehouse
mkdir -p data media/book-covers
# .env anlegen (siehe unten)
docker compose up -d --build
```

### .env

```
JWT_SECRET=<openssl rand -hex 32>
PORT=3000
```

### Update nach Code-Änderungen

```bash
git pull
docker compose up -d --build
```

Die `data/`- und `media/`-Verzeichnisse bleiben dabei unberührt.
