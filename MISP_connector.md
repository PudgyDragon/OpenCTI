<h1>MISP Connector for OpenCTI</h1>
<p>Self explanatory. These are the settings I used for a MISP connector that works with OpenCTI on a 
network behind a proxy.</p>

<h2>Configurations</h2>
<p>Before you start, make a user in OpenCTI in the GUI for your MISP connnector. Set it to be a connector 
  account and take note of the user token. Edit your docker-compose.yml file and use this template.</p>
<h4>NOTE: Change the OPENCTI_URL to your OpenCTI URL, and the MISP_URL the MISP instance you'll be 
  sending API calls to. The MISP_User_Token will be the user token of the connector account you just created. 
For the "import from date", choose one that isn't too far back. Your CPU will thank you.</h4>
<pre><code>
  vim /opt/OpenCTI/docker/docker-compose.yml
      MISP
      connector-misp:
          image: opencti/connector-misp:latest
          environment:
              - OPENCTI_URL=https://opencti-fqdn
              - OPENCTI_TOKEN=MISP_User_Token
              - CONNECTOR_ID=${CONNECTOR_MISP_ID}
              - CONNECTOR_NAME=MISP
              - CONNECTOR_SCOPE=misp
              - CONNECTOR_LOG_LEVEL=info
              - CONNECTOR_DURATION_PERIOD=PT5M
              - MISP_URL=https://misp-url
              - MISP_KEY=Your-MISP-API-Key
              - MISP_SSL_VERIFY=false
              - MISP_CREATE_REPORTS=true
              - MISP_CREATE_INDICATORS=true
              - MISP_CREATE_OBSERVABLES=true
              - MISP_CREATE_OBJECT_OBSERVABLES=true
              - MISP_CREATE_TAGS_AS_LABELS=true
              - MISP_REPORT_TYPE=misp-event
              - MISP_IMPORT_FROM_DATE=2026-02-01
              - MISP_IMPORT_DISTRIBUTION_LEVELS=0,1,2,3
              - MISP_IMPORT_THREAT_LEVELS=1,2,3,4
              - HTTP_PROXY=http://proxy:port
              - HTTPS_PROXY=http://proxy:port
              - NO_PROXY=localhost,127.0.0.0/8,172.0.0.0/8,opencti,rabbitmq,redis,elasticsearch,minio,your-fqdn,your-ipv4
              - http_proxy=http://proxy:port
              - https_proxy=http://proxy:port
              - no_proxy=localhost,127.0.0.0/8,172.0.0.0/8,opencti,rabbitmq,redis,elasticsearch,minio,your-fqdn,your-ipv4
          restart: always
          depends_on:
              opencti:
                  condition: service_healthy
</code></pre>
<p>Once you save your configurations, stop (if you haven't already) and start your container again</p>
<pre><code>
  docker compose down && docker compose up -d
</code></pre>
<p>Login to your GUI and check that it's ingesting.</p>

