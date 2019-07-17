## Harbor Installation and Configuartion Guid

### Table of Contents
1. [Offline installer](#1)
2. [Prerequisites for the target host](#3)
3. [Prerequisites softwares](#3)
4. [Installation Steps](#4)
5. [Register Docker Registry on CF](#5)

# <div id='1'/> 1. Offline installer
Harbor 설치 방법은 online 과 offline 으로 나뉜다.<br>
 - online installer  : installer가 Docker hub를 다운로드하기 때문에 installer 크기는 매우 작다.
 - offline installer : 호스트가 인터넷에 연결되어 있지 않을 때 이 방법을 사용한다. 설치 프로그램에는 미리 빌드 된 이미지가 포함되어 크기가 크다.

현재 설치된 Harbor Release 버전은 v1.6.2를 기준으로 가이드를 작성하였다.<br>



# <div id='2'/> 2. Prerequisites for the target host

Harbor는 여러 Docker 컨테이너로 배포되므로 Docker를 지원하는 모든 Linux 배포에 배포 할 수 있습니다. 대상 호스트는 Python, Docker 및 Docker Compose가 설치되어 있어야합니다.

### Hardware
<table>
  <tr>
    <th>Resource</th>
    <th>Capacity</th>
    <th>Descrpition</th>
  </tr>
  <tr>
    <td>CPU</td>
    <td>minimal 2 CPU</td>
    <td>4 CPU is preferred</td>
  </tr>
  <tr>
    <td>Mem</td>
    <td>minimal 4GB</td>
    <td>8GB is preferred</td>
  </tr>
  <tr>
    <td>Disk</td>
    <td>minimal 40GB</td>
    <td>160GB is preferred</td>
  </tr>
</table>

### Software
<table>
  <tr>
    <th>Software</th>
    <th>Version</th>
    <th>Description</th>
  </tr>
  <tr>
    <td>Python</td>
    <td>version 2.7 or higher</td>
    <td>기본적으로 파이썬 인터프리터가 설치되지 않은 리눅스 배포판 (Gentoo, Arch)에 파이썬을 설치해야 할 수도 있습니다</td>
  </tr>
  <tr>
    <td>Docker engine</td>
    <td>version 1.10 or higher</td>
    <td>설치 지침은 다음을 참조하십시오. https://docs.docker.com/engine/installation/</td>
  </tr>
  <tr>
    <td>Docker Compose</td>
    <td>version 1.6.0 or higher</td>
    <td>설치 지침은 다음을 참조하십시오. https://docs.docker.com/compose/install/</td>
  </tr>
  <tr>
    <td>Openssl</td>
    <td>latest is preferred</td>
    <td>Harbor에 대한 인증서 및 키 생성</td>
  </tr>
</table>

### Network ports
<table>
  <tr>
    <th>Port</th>
    <th>Protocol</th>
    <th>Descrpition</th>
  </tr>
  <tr>
    <td>443</td>
    <td>HTTPS</td>
    <td>Harbor UI 및 API는이 포트에서 https 프로토콜에 대한 요청을 수락합니다.</td>
  </tr>
  <tr>
    <td>4443</td>
    <td>HTTPS</td>
    <td>Harbord에 대한 Docker Content Trust 서비스 연결, Notary가 활성화 된 경우에만 필요.</td>
  </tr>
  <tr>
    <td>80</td>
    <td>HTTP</td>
    <td>Harbor UI 및 API는이 포트에서 http 프로토콜에 대한 요청을 수락합니다.</td>
  </tr>
</table>


# <div id='3'/> 3. Prerequisites softwares

##### 1. Pythone 설치

```
$ sudo add-apt-repository ppa:jonathonf/python-3.6
$ sudo apt-get update
$ sudo apt-get install python3.6
```



##### 2. Docker engine 설치

###### Step 1 - Installing Docker

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu   $(lsb_release -cs) stable"
$ sudo apt-get update
$ apt-cache policy docker-ce
$ sudo apt-get install -y docker-ce
$ sudo systemctl status docker

$ sudo systemctl status docker

```

```
Output
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2018-10-18 20:28:23 UTC; 35s ago
     Docs: https://docs.docker.com
 Main PID: 13412 (dockerd)
   CGroup: /system.slice/docker.service
           ├─13412 /usr/bin/dockerd -H fd://
           └─13421 docker-containerd --config /var/run/docker/containerd/containerd.toml
```

###### Step 2 - Executing the Docker Command Without Sudo

```
$ sudo usermod -aG  docker ${USER}
$ su - ${USER}
$ id -nG
$ sudo usermod -aG docker ${USER}
```



##### 3. Docker Compose 설치

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

$ docker-compose --version
  docker-compose version 1.23.1, build 1719ceb
```



# <div id='4'/> 4. Installation Steps

설치 단계는 다음과 같이 요약됩니다.

SSL 키의 경로. 프로토콜이 https로 설정된 경우에만 적용됩니다.

#### 1. Download the installer 

 -  offline installer 바이너리 파일([release](https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.2.tgz))을 다운로드 한다.
    
 -  패키지를 추출하려면 tar 명령을 사용하십시오.
    
        $ tar xvf harbor-offline-installer-1.6.2.tgz

#### 2. https를 사용 할 경우 인증서를 생성한다.(optional) 

 - cert 디렉토리 생성

      ```
      $ mkdir ~/cert
      $ cd ~/cert
      ```

 - Authority 인증서 생성

      ```
       $ openssl genrsa -out ca.key 4096
       $ openssl req -x509 -new -nodes -sha512 -days 3650 \
          -subj "/C=KR/ST=Seoul/L=Seoul/O=crossent/OU=paasxpert/CN={yourdomain}" \
          -key ca.key \
          -out ca.crt
      ```

        

 - Server 인증서 생성

   - Create your own Private Key:

            $ openssl genrsa -out yourdomain.com.key 4096

   - Generate a Certificate Signing Request:
   
            $ openssl req -sha512 -new \
                -subj "/C=KR/ST=Seoul/L=Seoul/O=crossent/OU=paasxpert/CN={yourdomain}" \
                 -key yourdomain.com.key \
                 -out yourdomain.com.csr 

  - 구성 및 설정 

    - Docker에 대한 서버 인증서, 키 및 CA 구성

        ```
        $ cat > v3.ext <<-EOF
          authorityKeyIdentifier=keyid,issuer
          basicConstraints=CA:FALSE
          keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
          extendedKeyUsage = serverAuth 
          subjectAltName = @alt_names
              
          [alt_names]
          DNS.1=yourdomain.com
          DNS.2=yourdomain
          DNS.3=hostname
          EOF
          
        $ openssl x509 -req -sha512 -days 3650 \
          -extfile v3.ext \
          -CA ca.crt -CAkey ca.key -CAcreateserial \
          -in yourdomain.com.csr \
          -out yourdomain.com.crt
        
        $ openssl x509 -inform PEM -in yourdomain.com.csr -out yourdomain.com.crt
        ```

    - Docker 용 yourdomain.com.crt, yourdomain.com.key 및 ca.crt를 배포합니다.

            $ cp yourdomain.com.crt /etc/docker/certs.d/yourdomain.com/
            $ cp yourdomain.com.key /etc/docker/certs.d/yourdomain.com/
            $ cp ca.crt /etc/docker/certs.d/yourdomain.com/

   - 다음은 생성된 인증서 구성을 보여줍니다.
        ​       

        ```
         /etc/docker/certs.d/
             └── yourdomain.com:port   
                ├── yourdomain.com.cert  <-- Server certificate signed by CA
                ├── yourdomain.com.key   <-- Server key signed by CA
                └── ca.crt               <-- Certificate authority that signed the registry certificate
        ```


#### 3. Configure harbor.cfg      

 - harbor.cfg 파일을 편집하고 호스트 이름과 프로토콜을 업데이트하고 속성 ssl_cert 및 ssl_cert_key를 업데이트합니다.

      ```
      #set hostname
      hostname = yourdomain.com:port
      #set ui_url_protocol
      ui_url_protocol = https
      ......
      #The path of cert and key files for nginx, they are applied only the protocol is set to https 
      
      ssl_cert = /data/cert/yourdomain.com.crt
      ssl_cert_key = /data/cert/yourdomain.com.key  
      ```

 - <b>hostname</b> : UI 및 레지스트리 서비스에 액세스하는 데 사용되는 대상 호스트의 호스트 이름입니다. 대상 컴퓨터의 IP 주소 또는 FQDN (정규화 된 도메인 이름)이어야합니다 (예 : 192.168.1.10 또는 reg.yourdomain.com). 호스트 이름에 localhost 또는 127.0.0.1을 사용하지 마십시오. 레지스트리 서비스를 외부 클라이언트가 액세스 할 수 있어야합니다.

 - <b>ui_url_protocol</b> : 
    (http 또는 https, 기본값은 http) UI 및 토큰 / 알림 서비스에 액세스하는 데 사용되는 프로토콜입니다. 인증서를 사용하는 경우 매개 변수는 https 여야합니다. 기본적으로는 http입니다. https 프로토콜을 설정하려면 [Configuring Harbor with HTTPS Access](https://github.com/goharbor/harbor/blob/master/docs/configure_https.md)을 참조하십시오.
  - <b>ssl_cert</b> : SSL 인증서의 경로. 프로토콜이 https로 설정된 경우에만 적용됩니다.
  - <b>ssl_cert_key</b> : SSL 키의 경로. 프로토콜이 https로 설정된 경우에만 적용됩니다.


#### 4. Run install.sh to install and start Harbor;  
 - Finishing installation and starting Harbor

      ```
      $ sudo ./install.sh
      ```

 - Generate configuration files for Harbor:

      ```
      $ sudo ./prepare
      ```

 - If Harbor is already running, stop and remove the existing instance. Your image data remain in the file system

      ```
      $ docker-compose down -v
      ```

 - Restart Harbor:

      ```
      $ docker-compose up -d
      $ docker login yourdomain.com
      ```

- Docker registry-insecure 등록

     ```
     $ sudo vi /etc/docker/daemon.json
     
     
     {"insecure-registries" : [ "10.20.1.7" ]}
     ```


#### 5. Set HAProxy

- HAProxy 설치

         $ sudo apt-get -y install haproxy

- harbor에 접속할 인증서 업데이트

         #harbor/common/config/nginx/cert 위치에 있는 cert키를 복사하여 haproxy를 설정 할 vm에 복사
         $ sudo vi /usr/local/share/ca-certificates/server.crt
         
         #/etc/ssl/private/server.pem 키 생성(yourdomain.com.crt와 yourdomain.com.key 포함)
         $ sudo vi /usr/local/private/server.pem
         
         #haproxy vm에 인증서 업데이트
         $ sudo update-ca-certificates

- haproxy.cfg 편집
     ​         
     ​    $ vi /etc/haproxy/haproxy.cfg
     ​     
         frontend https_frontend
                  bind *:80
                  bind *:443 ssl crt /etc/ssl/private/server.pem
                  http-request add-header X-Forwarded-Proto https if { ssl_fc }
                  option httpclose
                  default_backend web_server
         
         frontend https_frontend2
                  bind *:5000
                  bind *:443 ssl crt /etc/ssl/private/server.pem
                  http-request add-header X-Forwarded-Proto https if { ssl_fc }
                  option httpclose
                  default_backend web_server2
         
         backend web_server
                  mode http
                  balance roundrobin
                  server web2 10.10.1.15:443 check ssl verify none
                  http-request add-header X-Forwarded-Proto https if { ssl_fc }
                  server s1 10.10.1.15:80 check cookie s1
         
         backend web_server2
                  mode http
                  balance roundrobin
                  server web2 10.10.1.15:443 check ssl verify none
                  http-request add-header X-Forwarded-Proto https if { ssl_fc }
                  server s1 10.10.1.15:5000 check cookie s1

- haproxy.cfg 파일에 오류가 있는지 확인

         $ sudo haproxy -f /etc/haproxy/haproxy.cfg -c
         $ sudo service haproxy restart

- 접속 
     ​             

         https://101.55.50.205



#### 6. Harbor REST API via Swagger

- Download *prepare-swagger.sh* and *swagger.yaml* under the *docs* directory to your local Harbor directory, e.g. **~/harbor**.

  ```
  $ wget https://raw.githubusercontent.com/goharbor/harbor/master/docs/prepare-swagger.sh https://raw.githubusercontent.com/goharbor/harbor/master/docs/swagger.yaml
  
  ```

- Edit the script file *prepare-swagger.sh*.

  ```
  $ vi prepare-swagger.sh
  ```

- Change the SCHEME to the protocol scheme of your Harbor server.

  ```
  SCHEME=https
  SERVER_IP=<HARBOR_SERVER_DOMAIN(HAProxy DOMAIN)>
  ```

- Change the file mode.

  ```
  $ chmod +x prepare-swagger.sh
  ```

- Run the shell script. It downloads a Swagger package and extracts files into the *../static* directory.

  ```
  $ ./prepare-swagger.sh
  ```

- Edit the *docker-compose.yml* file under your local Harbor directory.

  ```
  $ vi docker-compose.yml
  
  ...
  ui:
    ... 
    volumes:
      - ./common/config/ui/app.conf:/etc/core/app.conf:z
      - ./common/config/ui/private_key.pem:/etc/core/private_key.pem:z
      - /data/secretkey:/etc/core/key:z
      - /data/ca_download/:/etc/core/ca/:z
      ## add two lines as below ##
      - ../src/ui/static/vendors/swagger-ui-2.1.4/dist:/harbor/static/vendors/swagger
      - ../src/ui/static/resources/yaml/swagger.yaml:/harbor/static/resources/yaml/swagger.yaml
      ...
  ```

- Recreate Harbor containers

  ```
  $ docker-compose down -v && docker-compose up -d
  ```





# <div id='5'>5. Register docker registry on CF

#### 1. cf-deployment.yml 수정

   ```
   $ cd workspace/cf
   $ vi cf-deployment/cf-deployment.yml
   ```

   ```
   - name: garden
       properties:
         garden:
           insecure_docker_registry_list: ## 추가
           - 10.20.1.7 ## harbor hostname 추가
           cleanup_process_dirs_on_wait: true
           debug_listen_address: 127.0.0.1:17019
           default_container_grace_time: 0
           deny_networks:
           - 0.0.0.0/0
           destroy_containers_on_start: true
           network_plugin: /var/vcap/packages/runc-cni/bin/garden-external-networker
           network_plugin_extra_args:
           - --configFile=/var/vcap/jobs/garden-cni/config/adapter.json
           persistent_image_list:
           - /var/vcap/packages/cflinuxfs2/rootfs.tar
         grootfs:
           reserved_space_for_other_jobs_in_mb: 15360
         logging:
           format:
             timestamp: rfc3339
       release: garden-runc
   ```

   ```
    - name: cflinuxfs2-rootfs-setup
       properties:
         cflinuxfs2-rootfs:
           trusted_certs: |+
           -----BEGIN CERTIFICATE-----
             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
             -----END CERTIFICATE-----
   ```

   ```
   - name: rep
       properties:
         bpm:
           enabled: true
         containers:
           trusted_ca_certificates:
           - |
           -----BEGIN CERTIFICATE-----
             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
             -----END CERTIFICATE-----
   ```
