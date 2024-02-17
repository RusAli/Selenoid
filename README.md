#  Настройка и установка Selenoid
## 1. Установить Docker и прописать команды:
   - ```sh
       sudo useradd -aG docker $USER
     ```
   - ```sh
       sudo chmod 666 /var/run/docker.sock
     ```
   - ```sh
       sudo systemctl restart docker
     ```
  
## 2. Создаем папку в пути /home/user
   - ```sh
     mkdir selenoid
     ```
   - и помещаем в нее файл browsers.json с содержимым
  
    
```json
{
  "chrome": {
    "default": "121.0",
    "versions": {
      "121.0": {
        "image": "selenoid/chrome:121.0",
        "port": "4444",
        "path": "/"
      },
      "120.0": {
        "image": "selenoid/chrome:120.0",
        "port": "4444",
        "path": "/"
      }
    }
  },
  "opera": {
    "default": "106.0",
    "versions": {
      "106.0": {
        "image": "selenoid/opera:106.0",
        "port": "4444",
        "path": "/"
      },
      "105.0": {
        "image": "selenoid/opera:105.0",
        "port": "4444",
        "path": "/"
      }
    }
  }
}
```
    
## 3. Создаем network для 2-ух контейнеров
   - ```sh
     docker network create selenoid
     ```
   - ```sh
     docker network create selenoid2
     ```
## 4. Запускаем докер контейнеры

> [!NOTE]
> Указать созданный network и правильно указать путь до файла browsers.json
   - ```sh
      docker run -d --rm --network selenoid --name selenoid -p 4445:4444 -v /var/run/docker.sock:/var/run/docker.sock -v /home/user/selenoid/browsers.json:/etc/selenoid/browsers.json:ro aerokube/selenoid:1.11.2 -container-network=selenoid -limit 12
      ```
   - ```sh
      docker run -d --rm --network selenoid2 --name selenoid2 -p 4446:4444 -v /var/run/docker.sock:/var/run/docker.sock -v /home/user/selenoid/browsers.json:/etc/selenoid/browsers.json:ro aerokube/selenoid:1.11.2 -container-network=selenoid2 -limit 12
      ```
## 5. Предустановка для GGR
   - ```sh
     sudo apt update
     ```
   - ```sh
     sudo apt install apache2-utils
     ```
   - ```sh
     sudo mkdir -p /etc/grid-router/quota/
     ```
   - Переходим в папку grid-router и создаем htpasswd
      ```sh
     cd /etc/grid-router/
     ```
      ```sh
     sudo htpasswd -bc /etc/grid-router/users.htpasswd test test-password
     ```
  - Создаем в папке grid-router файл test.xml ( test это имя пользователя из htpasswd)
  
    ```sh
     sudo vim /etc/grid-router/quota/test.xml
     ```

    С содержанием:

    ```xml
          <qa:browsers
	          xmlns:qa="urn:config.gridrouter.qatools.ru">
	          <browser name="chrome" defaultVersion="121.0">
		          <version number="121.0">
			          <region name="1">
				          <host name="172.18.0.1" port="4445" count="1"/>
				          <host name="172.19.0.1" port="4446" count="2"/>
			          </region>
		          </version>
		          <version number="120.0">
			          <region>
				          <host name="172.18.0.1" port="4445" count="1"/>
				          <host name="172.19.0.1" port="4446" count="2"/>
			          </region>
		          </version>
	          </browser>
          </qa:browsers>
    ```
> [!NOTE]
> host name - это Network-Gateway наших поднятых контейнеров selenoid и selenoid2 (docker inspect selenoid)
    
     
 ## 6. Поднимаем докер GGR

```sh
docker run -d --rm --name ggr -v /etc/grid-router/:/etc/grid-router:ro -p 4444:4444 aerokube/ggr:1.7.2 -guests-allowed -guests-quota "test" -verbose -quotaDir /etc/grid-router/quota
```
> [!NOTE]
> "test" - это имя пользователя из htpasswd

## 7. Поднимаем докер GGR-UI
```sh
docker run -d --rm --name ggr-ui -p 8888:8888 -v /etc/grid-router/quota/:/etc/grid-router/quota:ro aerokube/ggr-ui:1.2.0
```
> [!NOTE]
> Проверяем что ```curl -s http://127.0.0.1:8888/status``` возвращает json
> ```{"chrome":{"120.0":{},"121.0":{}},"opera":{"105.0":{},"106.0":{}}},"pending":0,"queued":0,"total":24,"used":0}```

## 8. Запускаем Selenoid-UI

Берем Network-Gateway из ggr-ui 
```sh
docker inspect ggr-ui
```

```sh
docker run -d --rm --name selenoid-ui -p 8080:8080 aerokube/selenoid-ui:1.10.11 --selenoid-uri http://172.17.0.1:8888
```
> [!NOTE]
> --selenoid-uri - это Network-Gateway из ggr-ui
 
 ## 9. Запускаем NGNIX

Создаем папку в директории /home/user/selenoid/

```sh
mkdir nginx
```
Создаем в созданной директории nginx файл с расширением .conf

```sh
mkdir nginx
```
Поднимаем nginx

```sh
docker run -d --rm --name nginx -v /home/user/nginx:/etc/nginx/conf.d:ro -d --network=host nginx
```



 
 

