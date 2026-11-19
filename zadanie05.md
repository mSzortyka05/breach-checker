# Zadanie 5

Skrót używany do testów: `bc45408273d7e1914b741c3407b0b559764f7876` (Kot-w-Kapeluszu-2026, nie ma go w próbce).

## 1. Healthcheck backendu

### docker compose up -d --build

```
TODO: wynik do powtórzenia po podmianie docker-compose.yml
(od linii "Container ... Starting", z Waiting / Healthy przy db i przy backend)
```

### docker compose ps

```
TODO: wynik do powtórzenia, przy db i backend powinno być (healthy)
```

## 2. Trwałość bazy

### Po INSERT

```
PS> docker compose exec db psql -U breach -d breachdb -c "INSERT INTO leaked_hashes VALUES ('bc45408273d7e1914b741c3407b0b559764f7876');"
TODO: wynik (INSERT 0 1)

PS> curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '{\"hash\":\"bc45408273d7e1914b741c3407b0b559764f7876\"}'
TODO: odpowiedź API
```

### Po docker compose down + docker compose up -d

```
PS> docker compose down
TODO: wynik

PS> docker compose up -d
[+] Running 4/4
 ✔ Network breach-checker_default       Created    0.0s
 ✔ Container breach-checker-db-1        Healthy    0.1s
 ✔ Container breach-checker-backend-1   Started    0.1s
 ✔ Container breach-checker-frontend-1  Started    0.1s

PS> curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '{\"hash\":\"bc45408273d7e1914b741c3407b0b559764f7876\"}'
{"found":true}
```

### Po docker compose down -v + docker compose up -d

```
PS> docker compose down -v
[+] Running 5/5
 ✔ Container breach-checker-frontend-1  Removed    0.6s
 ✔ Container breach-checker-backend-1   Removed   10.4s
 ✔ Container breach-checker-db-1        Removed    0.4s
 ✔ Volume breach-checker_pgdata         Removed    0.0s
 ✔ Network breach-checker_default       Removed    0.3s

PS> docker compose up -d
[+] Running 5/5
 ✔ Network breach-checker_default       Created    0.0s
 ✔ Volume "breach-checker_pgdata"       Created    0.0s
 ✔ Container breach-checker-db-1        Healthy    0.1s
 ✔ Container breach-checker-backend-1   Started    0.1s
 ✔ Container breach-checker-frontend-1  Started    0.1s

PS> curl.exe -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '{\"hash\":\"bc45408273d7e1914b741c3407b0b559764f7876\"}'
{"found":false}
```

### Wnioski

TODO: 2-3 zdania własnymi słowami - dlaczego skrót przetrwał `down`, a nie przetrwał `down -v`; dlaczego po `down -v` w bazie znów są 89 skróty z próbki.

## 3. Skalowanie

### docker compose ps (3 repliki)

```
TODO: wynik docker compose ps po docker compose up -d --scale backend=3
```

### docker compose exec frontend nslookup backend

```
Server:         127.0.0.11
Address:        127.0.0.11:53

Non-authoritative answer:

Non-authoritative answer:
Name:   backend
Address: 172.26.0.4
Name:   backend
Address: 172.26.0.3
Name:   backend
Address: 172.26.0.5
```

### Wysłanie 20 zapytań

```
PS> 1..20 | ForEach-Object { curl.exe -s -X POST http://localhost:8080/api/check -H "Content-Type: application/json" -d '{\"hash\":\"bc45408273d7e1914b741c3407b0b559764f7876\"}' }
{"found":false}
... (20 x {"found":false})
```

### Logi przed restartem frontendu

```
TODO: kilka linii POST /check sprzed docker compose restart frontend
```

### Logi po restarcie frontendu

```
PS> docker compose logs backend | Select-String "POST /check"
breach-checker-backend-1  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
breach-checker-backend-3  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
breach-checker-backend-3  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
breach-checker-backend-3  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
breach-checker-backend-1  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
breach-checker-backend-2  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
breach-checker-backend-2  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
breach-checker-backend-2  | 172.26.0.6 - - [19/Nov/2026 08:14:40] "POST /check HTTP/1.1" 200 -
```

TODO: sprawdzić, czy te linie na pewno pochodzą z zapytań po restarcie frontendu.

### Wnioski

TODO: 2-3 zdania własnymi słowami - dlaczego przed restartem frontendu ruch trafiał do jednej repliki; dlaczego skalowanie byłoby niemożliwe przy `ports: "5000:5000"`.