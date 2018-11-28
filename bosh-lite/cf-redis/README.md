# Redis Installation Guid on Bosh-Lite

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
    <td>cf-redis</td>
    <td></td>
  </tr>
  <tr>
    <td>routing</td>
    <td>0.180.0</td>
  </tr>
  <tr>
    <td>bpm</td>
    <td>0.12.2</td>
  </tr>
  <tr>
    <td>syslog</td>
    <td>v11.4.0</td>
  </tr>
</table>

#### Stemcell
<table>
  <tr>
    <th>스템셀 명</th>
    <th>버전</th>
  </tr>
  <tr>
    <td>ubuntu-trusty</td>
    <td>3586.40</td>
  </tr>
</table>

# <div id='2'/> 2. Deployment

- cf-redis-release clone
    
        $ mkdir -p ~/workspace/services
        $ cd ~/workspace/services
        
        $ git clone https://github.com/pivotal-cf/cf-redis-release
        $ cd ~/workspace/services/cf-redis-release
        $ git submodule update --init --recursive
        
- edit vars.yml for cf-redis 
 
        $ vi manifest/vars-lite.yml
        
            ---
            deployment_name: cf-redis  # name of your cf-redis deployment e.g. `cf-redis`
            broker_name: cf-redis-broker
            service_name: p-redis
            dedicated_vm_plan_id: 48b35349-d3de-4e19-bc4a-66996ae07766  # generate uuid, i.e. `uuidgen`
            service_id: 7aba7e52-f61b-4263-9de1-14e9d11fb67d  # generate different uuid, i.e. `uuidgen`
            shared_vm_plan_id: 78bf886c-bc50-4f31-a03c-cb786a158286  # generate different uuid, i.e. `uuidgen`
            dedicated_node_count: 1
            dedicated_nodes_ips: [10.20.5.20] # add a list of static ips from `default_network` in your
                                 # cloud-config with length `dedicated_node_count`
            director_uuid: *필수 입력 # UUID from `bosh -e bosh env`
            default_vm_type: small-highmem  # some vm_type from your cloud-config
            default_persistent_disk_type: 5GB # some disk_type from your cloud-config
            default_network: default  # some network from your cloud-config
            default_az: z1 # some az from your cloud-config
            stemcell_os: ubuntu-trusty
            stemcell_version: "3586.40"
            cf_deployment_name: cf # name of your cf deployment
            system_domain: msxpert.co.kr
            apps_domain: msxpert.co.kr
            broker_password: *필수 입력 # make one up
            broker_username: admin  # make one up
            cf_username: admin
            cf_password: *필수 입력  # `bosh -e bosh int ../../cf/env-repo/deployment-vars.yml --path /cf_admin_password` where
                          # `secrets/cf-creds.yml` is cf's vars-store
            syslog_endpoint_host: ""
            syslog_endpoint_port: ""

- update-cloud-config
        
        $ bosh -e bosh ucc cloud-config.yml
        
- cf-redis deploy
 
        $ bosh -e bosh-lite -d cf-redis deploy manifest/deployment.yml \
           --vars-file manifest/vars-lite.yml
           
- service-broker 등록           
           
        $ cf create-service-broker cf-redis-broker admin redis-secret http://cf-redis-broker.msxpert.co.kr
               
- 서비스 접근 등록
    
        $ cf enable-service-access p-redis