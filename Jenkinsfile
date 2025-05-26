pipeline {
    agent any

    environment {
        ELASTIC_USER = "elastic"
        ELASTIC_PASS = "PlnLz35OqHQ1UAOLqo8b"
        KIBANA_HOST  = "http://localhost:5601"     // Change to https:// if Kibana uses HTTPS
        ES_HOST      = "https://localhost:9200"    // Elasticsearch uses HTTPS
    }

    stages {

        stage('Deploy Node-RED Flow') {
            steps {
                sh '''
                echo "Deploying Node-RED Flow..."
                curl -X POST http://localhost:1880/flows \
                     -H "Content-Type: application/json" \
                     -d @node-red/flow.json
                '''
            }
        }

        stage('Deploy Logstash Config') {
            steps {
                sh '''
                echo "Deploying Logstash Config..."
                sudo cp logstash/mqtt_logstash.conf /etc/logstash/conf.d/
                sudo systemctl restart logstash
                '''
            }
        }

        stage('Register Elasticsearch Index Template') {
            steps {
                sh '''
                echo "Registering Elasticsearch Index Template..."
                curl -X PUT -u $ELASTIC_USER:$ELASTIC_PASS \
                     --insecure \
                     -H "Content-Type: application/json" \
                     -d @iot_index_template.json \
                     $ES_HOST/_index_template/iot-index-template
                '''
            }
        }

        stage('Register Kibana Index Pattern') {
            steps {
                sh '''
                echo "Registering Kibana Index Pattern..."
                curl -X POST -u $ELASTIC_USER:$ELASTIC_PASS \
                     --insecure \
                     -H "kbn-xsrf: true" \
                     -H "Content-Type: application/json" \
                     -d @kibana_template.json \
                     "$KIBANA_HOST/api/saved_objects/index-pattern/iot-*?overwrite=true"
                '''
            }
        }

        stage('Send Sample MQTT Data') {
            steps {
                sh '''
                echo "Checking if mosquitto_pub is installed..."
                if ! command -v mosquitto_pub > /dev/null; then
                    echo "ERROR: mosquitto_pub not found. Please install it using: sudo apt install mosquitto-clients"
                    exit 1
                fi

                echo "Publishing sample MQTT message..."
                mosquitto_pub -t "iot/sensor" -m '{"temperature": 23.4, "humidity": 55}'
                '''
            }
        }

        stage('Wait for Index Creation') {
            steps {
                sh '''
                echo "Waiting 10 seconds for Logstash to process data..."
                sleep 10

                echo "Checking created Elasticsearch indices..."
                curl -u $ELASTIC_USER:$ELASTIC_PASS --insecure $ES_HOST/_cat/indices?v
                '''
            }
        }
    }
}
