pipeline {
    agent any

    environment {
        ELASTIC_USER = "elastic"
        ELASTIC_PASS = "PlnLz35OqHQ1UAOLqo8b"
        KIBANA_HOST  = "http://localhost:5601"
        ES_HOST      = "http://localhost:9200"
    }

    stages {

        stage('Deploy Node-RED Flow') {
            steps {
                sh '''
                curl -X POST http://localhost:1880/flows \
                     -H "Content-Type: application/json" \
                     -d @node-red/flow.json
                '''
            }
        }

        stage('Deploy Logstash Config') {
            steps {
                sh '''
                sudo cp logstash/mqtt_logstash.conf /etc/logstash/conf.d/
                sudo systemctl restart logstash
                '''
            }
        }

        stage('Register Elasticsearch Index Template') {
            steps {
                sh '''
                curl -X PUT -u $ELASTIC_USER:$ELASTIC_PASS \
                     -H "Content-Type: application/json" \
                     -d @kibana/iot_index_template.json \
                     $ES_HOST/_index_template/iot-index-template
                '''
            }
        }

        stage('Register Kibana Index Pattern') {
            steps {
                sh '''
                curl -X POST -u $ELASTIC_USER:$ELASTIC_PASS \
                     -H "kbn-xsrf: true" \
                     -H "Content-Type: application/json" \
                     -d @kibana/kibana_template.json \
                     "$KIBANA_HOST/api/saved_objects/index-pattern/iot-*?overwrite=true"
                '''
            }
        }

        stage('Send Sample MQTT Data') {
            steps {
                sh '''
                mosquitto_pub -t "iot/sensor" -m '{"temperature": 23.4, "humidity": 55}'
                '''
            }
        }

        stage('Wait for Index Creation') {
            steps {
                sh '''
                echo "Waiting 10 seconds for Logstash to process data..."
                sleep 10
                curl -u $ELASTIC_USER:$ELASTIC_PASS $ES_HOST/_cat/indices?v
                '''
            }
        }
    }
}
