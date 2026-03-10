<h1>CISA KEV Connector for OpenCTI</h1>
<p>Self explanatory. These are the settings I used for a CISA KEV connector that works with OpenCTI on a 
network behind a proxy. And yes, I keep copy/pasting the basic instructions between connectors I've done. 
You can find the offical documentation for it here:</p>
<a href="https://github.com/OpenCTI-Platform/connectors/tree/master/external-import/cisa-known-exploited-vulnerabilities">CISA KEV Connector</a>

<h2>Configurations</h2>
<p>Before you start, make a user in OpenCTI in the GUI for your connnector. Set it to be a connector 
  account and take note of the user token. Edit your docker-compose.yml file and use this template.</p>
<h4>NOTE: Change the OPENCTI_URL to your OpenCTI URL. The OPENCTI_TOKEN will be the user token of the 
  connector account you just created. SPECIAL NOTE: For this connector, I had issues with the version. 
I had to specifically set the version to the version of OpenCTI I was running for it to work, so replace 
"your-version" in the <i>image</i> section with <i>your</i> OpenCTI version.</h4>
<pre><code>
  vim /opt/OpenCTI/docker/docker-compose.yml
      connector-cisa-known-exploited-vulnerabilities:
          image: opencti/connector-cisa-known-exploited-vulnerabilities:your-version
          environment:
              - OPENCTI_URL=https://opencti-fqdn
              - OPENCTI_TOKEN=CISA_User_Token
              - CONNECTOR_ID=${CONNECTOR_KEV_ID}
              - CONNECTOR_NAME=CISA KEV
              - CONNECTOR_SCOPE=cisa
              - CONNECTOR_LOG_LEVEL=info
              - CONNECTOR_DURATION_PERIOD=P1D
              - CISA_CATALOG_URL=https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json
              - CISA_CREATE_INFRASTRUCTURES=true
              - CISA_TLP=TLP:CLEAR
              - HTTP_PROXY=http://proxy:port
              - HTTPS_PROXY=http://proxy:port
              - NO_PROXY=localhost,127.0.0.0/8,172.0.0.0/8,opencti,rabbitmq,redis,elasticsearch,minio,your-fqdn,your-ipv4
              - http_proxy=http://proxy:port
              - https_proxy=http://proxy:port
              - no_proxy=localhost,127.0.0.0/8,172.0.0.0/8,opencti,rabbitmq,redis,elasticsearch,minio,your-fqdn,your-ipv4
          restart: always
</code></pre>
<p>If you didn't add the CONNECTOR_ID during the installation, add it to your .env file now. You'll need to generate
a GUID, I usually just use an online GUID generator because it's fast. Once you have it:</p>
<pre><code>
  vim /opt/OpenCTI/docker/.env
      CONNECTOR_KEV_ID=YOUR_GUID
  
</code></pre>
<p>Once you save your configurations, stop (if you haven't already) and start your container again</p>
<pre><code>
  docker compose down && docker compose up -d
  
</code></pre>
<p>Login to your GUI and check that it's ingesting.</p>
