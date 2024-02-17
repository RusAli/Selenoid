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
    
     
 ## 6. Предустановка для GGR
 

