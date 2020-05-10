https://golang.org/dl/

    wget -c https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
    tar xvf go1.14.2.linux-amd64.tar.gz
    sudo chown -R root:root ./go
    sudo mv go /usr/local
    sudo echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile
    source /etc/profile
    cd ~
    go version
    rm go1.14.2.linux-amd64.tar.gz

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-object-storage-server-using-minio-on-ubuntu-18-04
https://linuxhint.com/install_minio_ubuntu_1804/

    cd ~
    wget https://dl.min.io/server/minio/release/linux-amd64/minio

    sudo useradd --system minio --shell /sbin/nologin
    sudo usermod -L minio
    sudo chage -E0 minio

    sudo mv minio /usr/local/bin
    sudo chmod +x /usr/local/bin/minio
    sudo chown minio:minio /usr/local/bin/minio

    sudo touch /etc/default/minio
    sudo echo 'MINIO_ACCESS_KEY="minio"' >> /etc/default/minio
    sudo echo 'MINIO_VOLUMES="/usr/local/share/minio/"' >> /etc/default/minio
    sudo echo 'MINIO_OPTS="-C /etc/minio --address :9000"' >> /etc/default/minio
    sudo echo 'MINIO_SECRET_KEY="miniostorage"' >> /etc/default/minio

    sudo mkdir /usr/local/share/minio
    sudo mkdir /etc/minio

    sudo chown minio:minio /usr/local/share/minio
    sudo chown minio:minio /etc/minio

    cd ~

    wget https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service

    sed -i 's/User=minio-user/User=minio/g' minio.service
    sed -i 's/Group=minio-user/Group=minio/g' minio.service

    sudo mv minio.service /etc/systemd/system

    sudo systemctl daemon-reload
    sudo systemctl enable minio
    sudo systemctl start minio

    sudo systemctl status minio

    cd ~
