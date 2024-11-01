- **Подключение к удалённому хосту**
`ssh <host-name>@<host-address>`

- **Отключение соединения**
`Ctrl + D
`exit`
`~.`

- **Удаление fingerprint**
`ssh-keygen -R <host-address>`

- Передача файлов
`scp [options] [[user@]host1:]source_file_or_directory ... [[user@]host2:]destination`

![[scp_options.png]]
Пример:  
`scp -r BACK.png mu-pi@192.168.88.90:/home/mu-pi/TFT8_INDICATOR/source_code/resources`