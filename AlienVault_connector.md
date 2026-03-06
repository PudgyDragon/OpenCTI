<h1>AlienVault Connector for OpenCTI</h1>
<p>These are the settings I used for an AlienVault connector that works with OpenCTI on a 
network behind a proxy.</p>

<h2>Configurations</h2>
<p>Before you start, make a user in OpenCTI in the GUI for your connnector. Set it to be a connector 
  account and take note of the user token. Edit your docker-compose.yml file and use this template.</p>
<h4>NOTE: Change the OPENCTI_URL to your OpenCTI URL. The OPENCTI_TOKEN will be the user token of the 
  connector account you just created. For the "pulse start timestamp", choose one that isn't too far back.</h4>
<pre><code>
  vim /opt/OpenCTI/docker/docker-compose.yml
      connector-alienvault:
          image: opencti/connector-alienvault:latest
          environment:
              - OPENCTI_URL=https://opencti-url
              - OPENCTI_TOKEN=AlienVault_User_Token
              - CONNECTOR_ID=${CONNECTOR_ALIENVAULT_ID}
              - CONNECTOR_NAME=AlienVault
              - CONNECTOR_SCOPE=alienvault
              - CONNECTOR_LOG_LEVEL=error
              - CONNECTOR_DURATION_PERIOD=PT30M
              - ALIENVAULT_BASE_URL=https://otx.alienvault.com
              - ALIENVAULT_API_KEY=Your_api_key
              - ALIENVAULT_TLP=White
              - ALIENVAULT_CREATE_OBSERVABLES=true
              - ALIENVAULT_CREATE_INDICATORS=true
              - ALIENVAULT_PULSE_START_TIMESTAMP=2026-02-01T00:00:00
              - ALIENVAULT_REPORT_TYPE=threat-report
              - ALIENVAULT_REPORT_STATUS=New
              - ALIENVAULT_GUESS_MALWARE=false
              - ALIENVAULT_GUESS_CVE=false
              - ALIENVAULT_EXCLUDED_PULSE_INDICATOR_TYPES=FileHash-MD5,FileHash-SHA1
              - ALIENVAULT_ENABLE_RELATIONSHIPS=true
              - ALIENVAULT_ENABLE_ATTACK_PATTERNS_INDICATES=true
              - ALIENVAULT_DEFAULT_X_OPENCTI_SCORE=50
              - HTTP_PROXY=http://proxy:port
              - HTTPS_PROXY=http://proxy:port
              - NO_PROXY=localhost,127.0.0.0/8,172.0.0.0/8,opencti,rabbitmq,redis,elasticsearch,minio,your-fqdn,your-ipv4
              - http_proxy=http://proxy:port
              - https_proxy=http://proxy:port
              - no_proxy=localhost,127.0.0.0/8,172.0.0.0/8,opencti,rabbitmq,redis,elasticsearch,minio,your-fqdn,your-ipv4
          restart: always
          depends_on:
              opencti
</code></pre>
<p>Once you save your configurations, stop (if you haven't already) and start your container again</p>
<pre><code>
  docker compose down && docker compose up -d
</code></pre>
<p>Login to your GUI and check that it's ingesting.</p>
