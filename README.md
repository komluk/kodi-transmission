# Konfiguracja Transmission na Raspberry Pi

1. Na początku aktualizujemy listy pakietów.
```
sudo apt update
sudo apt upgrade
```

2.  Instalujemy demona torrentów transmission na Raspberry Pi.
```
sudo apt install transmission-daemon
```

# Instalacja serwera  Samba

1. Instalujemy pakiety wymagane do skonfigurowania Samby, uruchamiając następujące polecenie.
```
sudo apt-get install samba samba-common-bin
```
 
2. Tworzymy folder, który będziemy udostępniać.
```
mkdir /home/pi/media/shared
```

3. Teraz możemy udostępnić ten folder. Aby to zrobić, musimy zmodyfikować plik konfiguracyjny serwera Samba.
```
sudo nano /etc/samba/smb.conf
```

4. W tym pliku dodajemy na dole następujące elementy.
```
[shared]
path = /home/pi/media/shared
writeable=Yes
create mask=0777
directory mask=0777
public=no
```
Po zakończeniu musimy zapisać i wyjść, naciskając Ctrl + X, a następnie Y i ENTER.

5. Konfigurujemy użytkownika dla naszego udziału Samba na Raspberry Pi. Bez tego nie będziemy mogli nawiązać połączenia ze współdzielonym dyskiem sieciowym.
```
sudo smbpasswd -a pi
```

6. Restartujemy usługę samba, aby zaktualizować zmiany w konfiguracji.
```
sudo systemctl restart smbd
```

7. Ostatnią rzeczą, którą powinniśmy zrobić, zanim spróbujemy połączyć się z naszym udziałem Samba, jest odzyskanie lokalnego adresu IP naszego Raspberry Pi.
```
hostname -I
```

# Konfiguracja Transmission

1. Ponieważ Transmission jest uruchamiane automatycznie, musimy tymczasowo zatrzymać usługę, wykonując poniższe polecenie.
```
sudo systemctl stop transmission-daemon
```

2. Tworzymy dwa rózne foldery. Pierwszy to miejsce, w którym będziemy przechowywać pliki w trakcie pobierania, a drugi to miejsce, w którym będziemy przechowywać pobrane pliki.

Foldery nazywamy "torrent-inprogress" i "torrent-complete".

Tworzymy dwa foldery w katalogu który udostępniliśmy w "/media/shared/". Ten folder jest dostępny za pośrednictwem SAMBA servera
```
sudo mkdir -p /media/shared/torrent-inprogress
sudo mkdir -p /media/shared/torrent-complete
```

3. Nadajemy użytkownikowi "pi" uprawnienia do utworzonych folderów.
```
sudo chown -R pi:pi /media/shared/torrent-inprogress
sudo chown -R pi:pi /media/shared/torrent-complete
```

4. Wprowadzamy zmiany w pliku konfiguracyjnym.
```
sudo nano /etc/transmission-daemon/settings.json
```

5. Zmieniamy następujące opcje konfiguracji.
```
"incomplete-dir": "/media/shared/torrent-inprogress",
"download-dir": "/media/shared/torrent_complete",
"incomplete-dir-enabled": true,
"rpc-password": "Your_Password",
"rpc-username": "Your_Username",
"rpc-whitelist": "192.168.*.*",
```

# Zmiana użytkownika demona transmission

1. Modyfikujemy skrypt startowy demona Transmission, aby używał użytkownika "pi" zamiast domyślnego użytkownika "debian-transmission".
```
sudo nano /etc/init.d/transmission-daemon
```

2. W tym pliku edytujemy wiersz "USER=" aby demon był uruchamiany przez użytkownika "pi", a nie "debian-transmission", który jest domyślnie ustawiony.
```
USER=pi
```
Po zakończeniu musimy zapisać i wyjść, naciskając Ctrl + X, a następnie Y i ENTER.

3. Musimy także zmienić użytkownika w pliku usługi. W przeciwnym razie transmisja zostanie uruchomiona przez użytkownika "debian-transmission".
```
sudo nano /etc/systemd/system/multi-user.target.wants/transmission-daemon.service
```

W tym pliku musimy zmienić linię "User =", aby wskazywała na użytkownika "pi".
```
user=pi
```
Po zakończeniu musimy zapisać i wyjść, naciskając Ctrl + X, a następnie Y i ENTER.

4. Teraz należy wykonać polecenie aby menedżer usług ponownie załadował wszystkie pliki konfiguracji usługi, W przeciwnym razie systemctl spróbuje użyć starszej wersji pliku usługi.
```
sudo systemctl daemon-reload
```

5. Ponieważ zmieniliśmy użytkownika z "debian-transmission" na "pi", należy zmienić uprawnienia do folderu 
"/etc/transmission-daemon"
```
sudo chown -R pi:pi /etc/transmission-daemon
```

6. Tworzymy katalog, w którym demon uzyska dostęp do pliku "setting.json".
```
sudo mkdir -p /home/pi/.config/transmission-daemon/
sudo ln -s /etc/transmission-daemon/settings.json /home/pi/.config/transmission-daemon/
sudo chown -R pi:pi /home/pi/.config/transmission-daemon/
```

7. Na koniec restartujemy usługę Transmission na naszym Raspberry Pi.
```
sudo systemctl start transmission-daemon
```
