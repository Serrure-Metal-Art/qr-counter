# QR Counter — GitHub Pages + Supabase

## Architecture

```
GitHub Pages (docs/index.html)     <- Frontend statique, hébergé par GitHub
       |  fetch()
Supabase Edge Functions            <- Backend serverless (Deno)
  /functions/v1/count              <- retourne { count: N }
  /functions/v1/scan               <- incrémente + redirige
       |  SQL
Supabase PostgreSQL                <- Base de données persistante
  table: counter (id=1, value=N)
```

---

## ETAPE 1 — Supabase : base de données

### 1.1 Créer le projet
1. Aller sur https://supabase.com -> "Start your project"
2. Se connecter avec GitHub
3. "New project" : Name = qr-counter, Region = West EU (Ireland)
4. Attendre ~2 min

### 1.2 SQL à exécuter (SQL Editor -> New query)

```sql
CREATE TABLE IF NOT EXISTS counter (
  id    INTEGER PRIMARY KEY,
  value BIGINT  NOT NULL DEFAULT 0
);

INSERT INTO counter (id, value) VALUES (1, 0)
ON CONFLICT (id) DO NOTHING;

CREATE OR REPLACE FUNCTION increment_counter()
RETURNS void LANGUAGE plpgsql AS $$
BEGIN
  UPDATE counter SET value = value + 1 WHERE id = 1;
END;
$$;

ALTER TABLE counter ENABLE ROW LEVEL SECURITY;
CREATE POLICY "read"   ON counter FOR SELECT USING (true);
CREATE POLICY "update" ON counter FOR UPDATE USING (true);
GRANT EXECUTE ON FUNCTION increment_counter() TO anon, authenticated, service_role;
```

### 1.3 Récupérer les informations

**Project REF** :
- Project Settings -> General -> "Reference ID" (ex: abcdefghijklmnop)

**Access Token** :
- https://supabase.com/dashboard/account/tokens -> "Generate new token"
- Copier (affiché une seule fois !)

---

## ETAPE 2 — GitHub : créer le dépôt

1. https://github.com/new -> nom: qr-counter
2. NE PAS initialiser avec README
3. Uploader tous les fichiers du projet (glisser-déposer ou git push)

---

## ETAPE 3 — GitHub : secrets

Settings -> Secrets and variables -> Actions -> "New repository secret"

| Secret                  | Valeur                          |
|-------------------------|---------------------------------|
| SUPABASE_PROJECT_REF    | votre Reference ID              |
| SUPABASE_ACCESS_TOKEN   | le token généré en étape 1.3    |

---

## ETAPE 4 — Modifier docs/index.html

Ligne ~160, remplacer :
  const SUPABASE_PROJECT_REF = "REMPLACE_PAR_TON_PROJECT_REF";
par votre vrai Project REF, puis commiter.

---

## ETAPE 5 — GitHub Pages

Settings -> Pages -> Source: "Deploy from a branch"
Branch: main — Folder: /docs -> Save

URL finale : https://TON_USERNAME.github.io/qr-counter/

---

## ETAPE 6 — Déployer les Edge Functions

Le GitHub Actions workflow se déclenche automatiquement au push.
Pour forcer : Actions -> Run workflow

Vérification : https://VOTRE_REF.supabase.co/functions/v1/count
Doit retourner : {"count":0}

---

## URLs finales

| Quoi             | URL                                                    |
|------------------|--------------------------------------------------------|
| Page principale  | https://TON_USERNAME.github.io/qr-counter/             |
| API count        | https://VOTRE_REF.supabase.co/functions/v1/count       |
| API scan (QR)    | https://VOTRE_REF.supabase.co/functions/v1/scan        |
