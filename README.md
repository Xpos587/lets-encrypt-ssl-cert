# Let's Encrypt ssl certificate
## Получаем подписанный SSL-сертификат от Let's Encrypt с помощью OpenSSL и Certbot.

Документация содержит пошаговые инструкции по получению подписанного SSL-сертификата от Let's Encrypt для веб приложения. В данной документации мы будем получать SSL сертификат для субдомена [api.unboundshare.ru:25009](https://api.unboundshare.ru:25009).

```
Примечание: 'Вы можете создать SSL сетификат на другой машине, а потом перенести на основную!'
```

### Генерация **CSR** (Запрос на подпись сертификата)
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout unboundshare.ru.key -out unboundshare.ru.csr
```

Эта команда создаст файл **unboundshare.ru.key**, содержащий приватный ключ, и файл **unboundshare.ru.csr**, содержащий CSR.

### Установка **Certbot**

Обновите пакеты на вашем сервере с помощью следующей команды:
```bash
sudo apt-get update
```

Установите *Certbot* с помощью следующей команды:
```bash
sudo apt-get install certbot
```

*Certbot* - это инструмент, который позволяет автоматически получать и управлять сертификатами Let's Encrypt.

### Получение сертификата с помощью **Certbot**
Выполните следующую команду, чтобы получить сертификат Let's Encrypt:

```bash
sudo certbot certonly --manual --preferred-challenges=dns -d unboundshare.ru -d *.unboundshare.ru
```

Эта команда запустит процесс взаимодействия с *Certbot* для проверки вашего домена и получения сертификата. Убедитесь, что ваш домен unboundshare.ru и все его поддомены ссылаются на рабочиe VDS/VPS.

*Certbot* предложит вам выбрать способ подтверждения владения доменом, например, через DNS-запись.

```
Примечание: 'Убедитесь, что вы соблюдаете все инструкции *Certbot*, чтобы успешно получить сертификат!'
```

После успешного завершения этого шага у вас должны быть два файла сертификата: **privkey.pem** и **fullchain.pem** в директории `/etc/letsencrypt/live/unboundshare.ru/`. Это приватный ключ и полный сертификат, соответственно.

### Настройка вашего приложения для использования SSL, напримере `Uvicorn + Fast API`

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()

@app.get('/')
async def root():
    return {'message': 'Hello World'}

if __name__ == '__main__':
    uvicorn.run(
        'main:app',
        host='0.0.0.0',
        port=25009,
        ssl_keyfile='./privkey.pem',
        ssl_certfile='./cert.pem',
    )
```