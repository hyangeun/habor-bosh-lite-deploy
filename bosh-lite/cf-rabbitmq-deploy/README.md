# Rabbitmq Installation Guid on Bosh-Lite
### Table of Contents
1. [Prerequisites](#1)
2. [Deployment](#2)

# <div id='1'/> 1. Prerequisites
#### Releases
<table>
  <tr>
    <th>릴리즈 명</th>
    <th>버전</th>
  </tr>
  <tr>
    <td>cf-rabbitmq-multitenant-broker</td>
    <td>38.0.0</td>
  </tr>
  <tr>
    <td>routing</td>
    <td>0.152.0</td>
  </tr>
  <tr>
    <td>cf-cli</td>
    <td>0.10.0</td>
  </tr>
</table>

#### Stemcell
<table>
  <tr>
    <th>스템셀 명</th>
    <th>버전</th>
  </tr>
  <tr>
    <td>ubuntu-trusty(warden)</td>
    <td>3586.40</td>
  </tr>
</table>

# <div id='2'/> 2. Deployment

- cf-rabbitmq-multitenant-broker-release git clone
    
        $ mkdir -p ~/workspace/services
        $ cd ~/workspace/services
        
        $ git clone https://github.com/pivotal-cf/cf-rabbitmq-multitenant-broker-release.git
        $ cd ~/workspace/services/cf-rabbitmq-multitenant-broker-release
     

- manifests 및 scripts 파일 수정
    
   - 해당 디렉토리에 manifests 및 scripts 파일을 다운받고 workspace/services/cf-rabbitmq-multitenant-broker-release 디렉토리에 해당 파일들을 overwrite한다.
        
          수정 된 파일: 
          - manifests/add-cf-rabbitmq.yml
          - manifests/cf-rabbitmq-broker-template.yml
          - scripts/ deploy-to-bosh-lite
          - scripts/interpolate-manifest-for-bosh-lite

- cf-rabbitmq-multitenant-broker 릴리즈 업로드
           
          $ bosh -e bosh upload-release --sha1 11bed83a75ad1feca3ccd90136f9f5381d984aa3 \
              https://bosh.io/d/github.com/pivotal-cf/cf-rabbitmq-multitenant-broker-release?v=38.0.0
          
- bosh-lite deploy 스크립트 실행 

          $ cd ~/workspace/services/cf-rabbitmq-multitenant-broker-release
          $ ./scripts/deploy-to-bosh-lite
          
- service broker 등록

         $ cf create-service-broker p-rabbitmq broker broker http://rabbitmq-multitenant-broker.msxpert.co.kr

- 서비스 접근 등록
    
        $ cf enable-service-access p-rabbitmq         