## 1. Endpoint GET /health

Odpowiedź z `/health`:
```text
{"status":"ok"}
```
Zawartość git logs backend
```text
172.21.0.1 - - [12/Nov/2026 07:58:00] "GET /health HTTP/1.1" 200 -
```


## 2. Kontener diagnostyczny healthcheck-probe

Wyniki trzech uruchomień sondy:
```text
\$ docker run --rm --network breach-net healthcheck-probe
backend: OK

\$ docker run --rm healthcheck-probe
backend: UNREACHABLE

\$ docker stop backend
backend
\$ docker run --rm --network breach-net healthcheck-probe
backend: UNREACHABLE
\$ docker start backend
backend


## 3. Trwałość danych

Wynik `cat /tmp/notatka.txt` po restarcie kontenera:
```text
notatka
```

po usunięciu i ponownym odpaleniu:
```text
cat: /tmp/notatka.txt: No such file or directory
```

Zawartość historia.txt z wolumenu breach-dane:
PS C:\Users\mateusz.szortyka\Downloads\breach-checker\breach-checker> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt"
PS C:\Users\mateusz.szortyka\Downloads\breach-checker\breach-checker> docker run --rm -v breach-dane:/dane alpine sh -c "date >> /dane/historia.txt; cat /dane/historia.txt"
Thu Nov 12 08:18:48 UTC 2026
Thu Nov 12 08:19:07 UTC 2026