# VečerkaPlus — Noční buzení řidiče

Standalone HTML aplikace pro noční buzení řidiče rozvozu. Polluje Supabase každých 10 s a při nové objednávce spustí hlasitý alarm.

## Spuštění

1. Zkopíruj `config.example.js` jako `config.js` a doplň hodnoty:
   ```js
   const SUPABASE_URL = 'https://TVUJ-PROJECT.supabase.co';
   const SUPABASE_KEY = 'TVUJ-ANON-KEY';
   ```
2. Otevři `vecerka-buzeni.html` přes Live Server (VS Code) nebo jiný lokální HTTP server.  
   Nelze otevřít přímo jako `file://` — prohlížeč blokuje načítání externích skriptů.

## Supabase nastavení

Tabulka `orders` musí mít RLS policy pro `SELECT` s rolí `anon`:
```sql
create policy "anon select orders"
on orders for select to anon using (true);
```

Schéma tabulky:
```
id          UUID
created_at  timestamptz
name        text
address     text
phone       text
payment     text
items       JSONB   -- [{name, qty, price, ...}]
total       integer
status      text    -- nová / přijatá / doručená / zrušená
```

## Jak to funguje

- Při načtení stránky se naplní `knownIds` z aktuálních objednávek — alarm se nespouští.
- Každých `POLL_SEC` sekund se dotáže Supabase na nové objednávky (status `nová`).
- Nová objednávka = spustí overlay přes celou obrazovku + opakovaný zvuk každé `ALARM_REPEAT_SEC` sekundy.
- **Beru to** — potvrdí objednávku, alarm zastaví, ID přidá do `knownIds`.
- **Odložit na 2 min** — snooze, alarm se nespustí znovu po dobu 2 minut.

## Konfigurace

| Konstanta         | Default | Popis                              |
|-------------------|---------|------------------------------------|
| `POLL_SEC`        | 10      | Interval pollování v sekundách     |
| `ALARM_REPEAT_SEC`| 3       | Jak často se opakuje zvuk alarmu   |
