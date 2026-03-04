<h1>OpenCTI Installation Guide</h1>
<p>As the title suggests, this is a guide for installing OpenCTI. The underlying OS for this install will be RHEL 9. 
I will cover a basic partition setup for this. Your organization may have a different setup. This guide 
also assumes that you are behind a corporate proxy. It took some trial and error to get it working, 
hopefully it will work for you too. I had a hard time finding guides for installing OpenCTI behind a proxy.</p>

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

<p>ACTUAL INSTALLATION COMING SOON, HOPEFULLY TOMORROW. RAN OUT OF TIME.</p>
