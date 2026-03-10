<h1>OpenCTI Installation Guide</h1>
<p>As the title suggests, this is a guide for installing OpenCTI. The underlying OS for this install will be RHEL 9. 
I will cover a basic partition setup for this. Your organization may have a different setup. This guide 
also assumes that you are behind a corporate proxy. It took some trial and error to get it working, 
hopefully it will work for you too. I had a hard time finding guides for installing OpenCTI behind a proxy. Becuase I'm terrible, unless specified (as in the OpenCTI installation section) these commands are going to be run as root (or prepend sudo to the command if you're better than me).</p>

<h2>Requirements</h2>
<p>OpenCTI requirements will vary based on your needs. You can find an overview at this URL </p>
<a href="https://docs.opencti.io/latest/deployment/overview/">OpenCTI Overview</a>
<p>For the purpose of laziness, and in case the url stops working</p>
<table>
  <tr>
    <th>Component</th>
    <th>CPU</th>
    <th>RAM</th>
    <th>Disk Type</th>
    <th>Disk Space</th>
  </tr>
  <tr>
    <th colspan="100">Dependencies</th>
  </tr>
  <tr>
    <td>ElasticSearch / OpenSearch</td>
    <td>2 Cores</td>
    <td>≥ 8GB</td>
    <td>SSD</td>
    <td>≥ 16GB</td>
  </tr>
  <tr>
    <td>Redis</td>
    <td>1 Core</td>
    <td>≥ 1GB</td>
    <td>SSD</td>
    <td>≥ 16GB</td>
  </tr>
  <tr>
    <td>RabbitMQ</td>
    <td>1 Core</td>
    <td>≥ 512MB</td>
    <td>Standard</td>
    <td>≥ 2GB</td>
  </tr>
  <tr>
    <td>S3 / MinIO</td>
    <td>1 Core</td>
    <td>≥ 128MB</td>
    <td>SSD</td>
    <td>≥ 16GB</td>
  </tr>
  <tr>
    <th colspan="100">Platform</th>
  </tr>
  <tr>
    <td>OpenCTI Core</td>
    <td>2 Cores</td>
    <td>≥ 8GB</td>
    <td>None (stateless)</td>
    <td></td>
  </tr>
  <tr>
    <td>Worker(s)</td>
    <td>1 Core</td>
    <td>≥ 128MB</td>
    <td>None (stateless)</td>
    <td></td>
  </tr>
  <tr>
    <td>Connector(s)</td>
    <td>1 Core</td>
    <td>≥ 128MB</td>
    <td>None (stateless)</td>
    <td></td>
  </tr>
  <tr>
    <td>XTM composer</td>
    <td>1 Core</td>
    <td>≥ 128MB</td>
    <td>None (stateless)</td>
    <td></td>
  </tr>
</table>
<p>RAID on SSD drives should be set up through your BIOS setup using the RAID configuration capability. 
If you have NVMe drives, you will set up the raid during the installation of RHEL. Configure raid to 
your hearts desire. I would recommend some type of RAID be set up to ensure your data doesn't get 
completely borked.</p>

<h2>RHEL 9 Installation</h2>
<p>Not going to go into much detail, just basics. Boot up your device using bootable media and follow 
the installation prompts.</p>
<h3>IMPORTANT</h3>
<p>If you wish to have true FIPS compliance, or set yourself up for it, ensure that you do the following: </p>
<pre>
  <code>
    Press e at the grub menu
    Add fips=1 after "quiet" or "rhbg"
    Ctrl + x to save and boot
    
  </code>
</pre>
<h3>IMPORTANT</h3>
<p>If you're following DISA STIG guides, don't choose a profile during installation. Certain settings 
(like selinux) will cause issues during the installation process. Make sure you choose the option for 
a server without a GUI (NOT minimalistic, however).</p>
<p>Partitioning can be set up to look similar to these. I separated opt in it's own group, but you can 
set it however you want</p>
<ul>
  <li>DATA
    <ul>
      <li>rootrhel (xfs)
        <ul>
          <li>/home</li>
          <li>/storetmp</li>
          <li>/var/log</li>
          <li>/var/log/audit</li>
          <li>/var/tmp</li>
        </ul>
      </li>
      <li>sda (standard)
        <ul>
          <li>/recovery</li>
        </ul>
      </li>
      <li>optrhel (xfs)
        <ul>
          <li>/opt</li>
        </ul>
      </li>
    </ul>
  </li>
  <li>SYSTEM
    <ul>
      <li>rootrhel (xfs)
        <ul>
          <li>/</li>
          <li>/tmp</li>
          <li>/var</li>
          <li>swap</li>
        </ul>
      </li>
      <li>sda (standard)
        <ul>
          <li>/boot/efi</li>
          <li>/boot</li>
        </ul>
      </li>
    </ul>
  </li>
</ul>

<h3>RHEL Done</h3>
<p>Once the installation is complete, move on to setting up Docker.</p>

<h2>Docker on RHEL</h2>
<p>I initially didn't read the document for this the right way, and it caused some issues. After you
get it set up with your RHEL satellite server, you 100% need to remove any pre-installed docker 
dependencies, which will be covered.</p>
<h3>Starter Proxy Settings</h3>
<p>If you haven't already, change your environment settings to use your proxy</p>
<pre><code>
  vim /etc/environment
      http_proxy="http://fqdn:port"
      https_proxy="http://fqdn:port"
      no_proxy="localhost, 127.0.0.1, whateva else"
      HTTP_PROXY="http://fqdn:port"
      HTTPS_PROXY="http://fqdn:port"
      NO_PROXY="localhost, 127.0.0.1, whateva else"
  
</code></pre>
<p>Logout of your current session and back in to use the new environment settings.</p>
<h3>Out with the Old, in with the New</h3>
<p>Delete anything docker related that may have been installed by your RHEL installation. I think 
it goes without saying, if you're running Docker in a corporate environement already and this is yet 
another application on the device, DON'T DO THE FOLLOWING!</p>
<pre><code>
  dnf remove docker \
      docker-client \
      docker-client-latest \
      docker-common \
      docker-latest \
      docker-latest-logrotate \
      docker-logrotate \
      docker-engine \
      podman \
      runc
  
</code></pre>
<p>Once that completes, install docker for realsies</p>
<pre><code>
  dnf -y install dnf-plugins-core
  dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
  
</code></pre>
<p>Once the docker repo has been created, you'll need to configure it to use your proxy settings 
and run dnf update and install the important bits, then enable it</p>
<pre><code>
  vim /etc/yum.repos.d/docker-ce.repo
      At the top of each entry, under the main header add:
      proxy=http://fqdn:port
  dnf update
  dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
  systemctl enable --now docker
  
</code></pre>
<p>Once enabled, create the http-proxy.conf file for docker to get your proxy settings from, along with 
the config.json file</p>
<pre><code>
  mkdir /etc/systemd/system/docker.service.d/
  vim /etc/systemd/system/docker.service.d/http-proxy.conf
      [Service]
      Environment="HTTP_PROXY=http://fqdn:port/"
      Environment="HTTPS_PROXY=http://fqdn:port/"
      Environment="NO_PROXY=localhost,127.0.0.1,whatevaelse"
  mkdir ~/.docker
  vim ~/.docker/config.json
      {
        "proxies": {
          "default": {
            "httpProxy": "http://fqdn:port",
            "httpsProxy": "http://fqdn:port",
            "noProxy": "localhost,127.0.0.0/8,172.0.0.0/8,whatevaelse
          }
        }
      }
  
</code></pre>
<h4>NOTE: OpenCTI uses the 172.0.0.0/8 IP space for its containers. For testing purposes I left the subnet 
block rather large. Play with it as you go and configure it to a smaller subnet for security purposes.</h4>
<p>Restart the proper services so these settings take effect</p>
<pre><code>
  systemctl daemon-reload
  systemctl restart docker
  
</code></pre>

<h3>SSL Certs</h3>
<p>Get your CA and Root cert setup out of the way now. If you're using custom CA certs, or you're unsure 
if yours are in the trust bundle</p>
<pre><code>
  vim /etc/pki/ca-trust/source/anchors/your_ca.pem
      Copy/Paste certificate contents and save
  vim /etc/pki/ca-trust/source/anchors/your_root.pem
      Copy/Paste certificate contents and save
  update-ca-trust
  
</code></pre>

<h2>OpenCTI Installation</h2>
<p>Last but not least, the installation of OpenCTI. I went back and forth between several guides for guidance 
and gleaning some practices they used. I settled on the main guide from Filigran themselves and one from 
the following URL</p>
<ul>
  <li>https://benheater.com/proxmox-running-opencti/</li>
</ul>

<h3>Dependencies</h3>
<p>Install a couple dependencies that OpenCTI needs, and change the max map count that it requires you to change</p>
<pre><code>
  dnf install git
  dnf install jq
  sysctl -w vm.max_map_count=1048575
  echo 'vm.max_map_count=1048575' | sudo tee --append /etc/sysctl.conf
  
</code></pre>

<h3>OpenCTI Service Account</h3>
<p>Make the directory your app will be in and create a service account that will be used for making changes to OpenCTI. Make its home directory the location that your app will be in. I named mine OpenCTI and placed it in the /opt directory, but you can do whatever you want</p>
<pre><code>
  mkdir /opt/OpenCTI
  useradd -r -s /bin/bash -m -d /opt/OpenCTI opencti_svc
  passwd opencti_svc
  usermod -a -G docker opencti_svc
  
</code></pre>
<p>In another terminal, login as your opencti_svc account.</p>

<h3>Clone Repo</h3>
<p>Change directories into your newly created OpenCTI directory and clone the repo. Once there, make a backup of it just in case one of us screws something up along the way</p>
<pre><code>
  cd /opt/OpenCTI
  git clone https://github.com/OpenCTI-Platform/docker.git
  cd docker
  cp docker-compose.yml docker-compose.yml.bak
  
</code></pre>

<h3>Configurations</h3>
<p>Start off the bat with HTTPS in mind. By default, OpenCTI uses port 8080. For one reason or another, you may not want to use 8080 as your port. The internal application will need to continue using 8080, but it can be mapped to 443 for external communication</p>
<pre><code>
  vim docker-compose.yml
      Change the setting in the opencti section for ports from
          {opencti_port}:8080
      to
          443:8080
  
</code></pre>
<p>Run the following to configure OpenCTI settings. Make changes where necessary (ie email, password, base url, etc)</p>
<pre><code>
  (cat << EOF
      OPENCTI_ADMIN_EMAIL=admin@fqdn
      OPENCTI_ADMIN_PASSWORD=ChangeMePlease
      OPENCTI_ADMIN_TOKEN=$(cat /proc/sys/kernel/random/uuid)
      OPENCTI_BASE_URL=https:fqdn
      OPENCTI_HEALTHCHECK_ACCESS_KEY=$(cat /proc/sys/kernel/random/uuid)
      MINIO_ROOT_USER=$(cat /proc/sys/kernel/random/uuid)
      MINIO_ROOT_PASSWORD=$(cat /proc/sys/kernel/random/uuid)
      RABBITMQ_DEFAULT_USER=guest
      RABBITMQ_DEFAULT_PASS=guest
      ELASTIC_MEMORY_SIZE=8G
      CONNECTOR_HISTORY_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_EXPORT_FILE_STIX_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_EXPORT_FILE_CSV_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_IMPORT_FILE_STIX_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_EXPORT_FILE_TXT_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_IMPORT_DOCUMENT_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_ANALYSIS_ID=$(cat /proc/sys/kernel/random/uuid)
      SMTP_HOSTNAME=localhost
      CONNECTOR_MITRE_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_IMPORT_EXTERNAL_REFERENCE_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_OPENCTI_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_IMPORT_FILE_YARA_ID=$(cat /proc/sys/kernel/random/uuid)
      XTM_COMPOSER_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_ALIENVAULT_ID=$(cat /proc/sys/kernel/random/uuid)
      CONNECTOR_MISP_ID=$(cat /proc/sys/kernel/random/uuid)
      OPENCTI_EXTERNAL_SCHEME=https
      OPENCTI_HOST=fqdn
      OPENCTI_PORT=443
      EOF
  ) > .env
    
</code></pre>
<p>Give it the ole up down to get these configurations going</p>
<pre><code>
  docker compose up -d && docker compose down
  
</code></pre>
<p>Edit the docker-compose.yml file and make the following changes, followed by another up down</p>
<pre><code>
  vim docker-compose.yml
      In the opencti section
          opencti:
              volumes:
                  -opencti_https:/certs
              environment:
                  - APP__HTTPS_CERT__KEY=/certs/opencti.key
                  - APP__HTTPS_CERT__CRT=/certs/opencti.crt
                  - APP__HTTPS_CERT__REJECT_UNAUTHORIZED=false
                  - HTTP_PROXY=http://fqdn:port
                  - HTTPS_PROXY=http://fqdn:port
                  - NO_PROXY=localhost,127.0.0.0/8,172.0.0.0/8,opencti,rabbitmq,redis,elasticsearch,minio,fqdn,IPv4
      In the bottom volumes section
          volumes:
              opencti_https:
      Any instance of http://opencti:8080
          https://fqdn
      Change the healthcheck for opencti to not verify SSL
          "wget", "--no-check-certificate", "https://fqdn/health?health_access_key=${OPENCTI_HEALTHCHECK_ACCESS_KEY}"
  docker compose up -d && docker compose down
  
</code></pre>
<h4>NOTE - wget does SSL verification with the healthcheck. For testing purposes I bypassed it with the no check option.</h4>
<p>This will set up your proxy settings for OpenCTI, and create the directory where you can put your servers SSL certificate and key for HTTPS. You will need to put your .key and .crt files in the following location and name them opencti.key and opencti.crt (or whatever you want as long as they're named the same in the docker-compose file we just changed)</p>
<pre><code>
  /var/lib/docker/volumes/docker_opencti_https/_data/
  
</code></pre>
<p>At this point you can now start the container and you should be able to navigate to your url</p>
<pre><code>
  docker compose up -d
  
</code></pre>

<h3>Extras</h3>
<p>If your hardware has the capability and you're not worried about clustering at the moment, you can try bumping up the number of workers you have. I wouldn't do too many though. I went from 3 to 5 for testing</p>
<pre><code>
  vim docker-compose.yml
      worker:
          replicas: 5
  
</code></pre>

<h2>Fin</h2>
<p>There are several connectors you can use within OpenCTI. I will cover AlienVault and MISP in separate guides of their own shortly, but it's past my bed time.</p>

