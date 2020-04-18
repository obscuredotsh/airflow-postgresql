pipeline {
    agent any
    parameters {
    string defaultValue: 'Spawner', description: '', name: 'AWS_KEY_PAIR', trim: false
    string defaultValue: 'subnet-04570bb34003b20f7', description: '', name: 'SUBNET', trim: false
    }

    stages {
    stage ('Launching Airflow Server') {
    steps {
    sh ''' 
    rm -rf *
    figlet Launching AirFlow server
    rm -rf workdir ; mkdir workdir ; cd workdir 
    cp -r /home/devops/spawner/step1/* .
    sed -i s/'Name: Batch Processing Machine'/'Name: Airflow'/g instance.yaml
    sed -i s/'Spawner-SG'/'airflow'/g instance.yaml
    
    ansible-playbook instance.yaml --extra-vars "AWS_KEY_PAIR=$AWS_KEY_PAIR ACCESS_KEY='$ak' SECRET_KEY='sk' SUBNET=$SUBNET" > output ; 
    cat output | grep msg | cut -d':' -f2 | cut -d'"' -f2 | head -n1 > /tmp/AirflowIPs
    
    cat /tmp/AirflowIPs
    '''
    }
    }
    stage ('Deploying Airflow as Webserver') {
    steps {
    sh   ''' 
    cd workdir
    sleep 60

    rsync -ravhP /home/devops/airflow/step2/ansible-airflow.tar.gz -e 'ssh -o StrictHostKeyChecking=no -i /home/ubuntu/pemfile/Spawner.pem' ubuntu@`cat /tmp/AirflowIPs`:~
    ssh -o StrictHostKeyChecking=no -i /home/ubuntu/pemfile/Spawner.pem ubuntu@`cat /tmp/AirflowIPs` tar xvf ansible-airflow.tar.gz
    ssh -o StrictHostKeyChecking=no -i /home/ubuntu/pemfile/Spawner.pem ubuntu@`cat /tmp/AirflowIPs` ansible-playbook ansible-airflow/airflow.yaml
    rm -rf *
    '''

    }
    }
    }
    }
