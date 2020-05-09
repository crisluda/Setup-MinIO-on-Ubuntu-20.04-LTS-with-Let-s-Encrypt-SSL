Install Minio on Ubuntu 18.04

https://www.digitalocean.com/community/tutorials/how-to-install-go-on-ubuntu-18-04

https://golang.org/dl/

    curl -O https://dl.google.com/go/go1.14.2.linux-amd64.tar.gz
    tar xvf go1.14.2.linux-amd64.tar.gz
    sudo chown -R root:root ./go
    sudo mv go /usr/local
    echo 'export GOPATH=$HOME/.go' >> ~/.profile
    echo 'export GOROOT=/usr/local/go' >> ~/.profile
    source ~/.profile
    export GOPATH=$HOME/.go
    export GOROOT=/usr/local/go
    cd /usr/local/bin/
    ln -s /usr/local/go/bin/go
    ln -s /usr/local/go/bin/gofmt
    cd ~
    rm go1.14.2.linux-amd64.tar.gz


https://docs.min.io/docs/minio-client-complete-guide.html

    cd ~
    apt install -y build-essential
    go get -d github.com/minio/mc
    cd ${GOPATH}/src/github.com/minio/mc
    make
    cd /usr/local/bin/
    ln -s /root/.go/src/github.com/minio/mc/mc

https://www.digitalocean.com/community/tutorials/how-to-set-up-an-object-storage-server-using-minio-on-ubuntu-18-04
https://linuxhint.com/install_minio_ubuntu_1804/

    cd ~
    wget https://dl.min.io/server/minio/release/linux-amd64/minio

    useradd --system minio --shell /sbin/nologin
    usermod -L minio
    chage -E0 minio

    mv minio /usr/local/bin
    chmod +x /usr/local/bin/minio
    chown minio:minio /usr/local/bin/minio

    touch /etc/default/minio
    echo 'MINIO_ACCESS_KEY="minio"' >> /etc/default/minio
    echo 'MINIO_VOLUMES="/usr/local/share/minio/"' >> /etc/default/minio
    echo 'MINIO_OPTS="-C /etc/minio --address :9000"' >> /etc/default/minio
    echo 'MINIO_SECRET_KEY="miniostorage"' >> /etc/default/minio

    mkdir /usr/local/share/minio
    mkdir /etc/minio

    chown minio:minio /usr/local/share/minio
    chown minio:minio /etc/minio

    cd ~

    curl -O https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service

    sed -i 's/User=minio-user/User=minio/g' minio.service
    sed -i 's/Group=minio-user/Group=minio/g' minio.service

    mv minio.service /etc/systemd/system

    systemctl daemon-reload
    systemctl enable minio
    systemctl start minio

    systemctl status minio

    cd ~

https://gist.github.com/fntlnz/cf14feb5a46b2eda428e000157447309

    cd ~

    sed "/^RANDFILE.*$ENV::HOME\/\.rnd/d" -i /etc/ssl/openssl.cnf

    export IP_ADDRESS=$(hostname -I)
    export DNS_ADDRESS=$(hostname -I)
    
    openssl genrsa -des3 -out rootCA.key 4096
    openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.crt
    openssl genrsa -out minio.key 4096
    openssl req -new -sha256 -key minio.key -subj "/C=US/ST=CA/O=MyOrg, Inc./CN=${DNS_ADDRESS}" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "\n[SAN]\nsubjectAltName=DNS:${DNS_ADDRESS},IP:${IP_ADDRESS}")) -out minio.csr

    openssl req -in minio.csr -noout -text

    openssl x509 -req -in minio.csr -CA rootCA.crt -CAkey rootCA.key -CAcreateserial -out minio.crt -extfile <(printf "\n[SAN]\nsubjectAltName=DNS:${DNS_ADDRESS},IP:${IP_ADDRESS}") -days 500 -sha256 -ext SAN -extensions SAN

    openssl x509 -in minio.crt -text -noout

    mv minio.crt /etc/minio/certs/public.crt
    mv minio.key /etc/minio/certs/private.key

    chown -R minio:minio /etc/minio

    cp rootCA.crt /etc/ssl/certs/

    systemctl restart minio
    journalctl -u minio -f -n 100

    mc config host add minio https://${IP_ADDRESS}:9000 minio miniostorage
    mc ls minio

    cd ~
    rm minio.csr
    # rm rootCA.srl
    # rm rootCA.key
    # rm rootCA.crt
