# Jenkins Logs to ELK using Filebeat

This project sets up Jenkins to send its logs to an ELK (Elasticsearch, Logstash, Kibana) stack using Filebeat.

---

## 1. Customize Jenkins Log Location

To customize the Jenkins log location, run:

```bash
sudo systemctl edit jenkins
```

Add the following:

```ini
[Service]
Environment="JENKINS_LOG=%L/jenkins/jenkins.log"
```

Then reload and restart Jenkins:

```bash
sudo systemctl daemon-reload
sudo systemctl restart jenkins
```

---

## 2. Install and Configure Filebeat

- Install Filebeat on the Jenkins server.

```bash
sudo apt install filebeat
```

- Configure Filebeat to track the Jenkins log:

Edit `/etc/filebeat/filebeat.yml` and add under `filebeat.inputs`:

```yaml
- type: log
  paths:
    - /var/log/jenkins/jenkins.log
```

- Set the output to Logstash or Elasticsearch in the same file:

```yaml
output.logstash:
  hosts: ["<logstash-server>:5044"]
```
or
```yaml
output.elasticsearch:
  hosts: ["http://<elasticsearch-server>:9200"]
```

- Start and enable Filebeat:

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

---

## 3. Setup ELK Stack

- Deploy or connect to an ELK stack (Elasticsearch, Logstash, Kibana).
- Ensure Logstash (if used) can parse incoming logs.
- Access Kibana to view Jenkins logs under Discover.

---

## 📌 Notes

- Make sure permissions allow Jenkins to write to the custom log file location.
- Adjust Filebeat modules if needed for better parsing.

