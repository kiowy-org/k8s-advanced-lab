# Installer les outils ETCD sur le cluster kubeadm

1. Rendez-vous [https://github.com/etcd-io/etcd/releases/latest](https://github.com/etcd-io/etcd/releases/latest) et suivez les instructions d'installation.

    _Exemple pour la [v3.5.14](https://github.com/etcd-io/etcd/releases/tag/v3.5.14)_

    ```shell
    ETCD_VER=v3.5.14
    
    # choose either URL
    GOOGLE_URL=https://storage.googleapis.com/etcd
    GITHUB_URL=https://github.com/etcd-io/etcd/releases/download
    DOWNLOAD_URL=${GOOGLE_URL}
    
    rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
    rm -rf /tmp/etcd-download-test && mkdir -p /tmp/etcd-download-test
    
    curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
    tar xzvf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-download-test --strip-components=1
    rm -f /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz
    
    /tmp/etcd-download-test/etcd --version
    /tmp/etcd-download-test/etcdctl version
    /tmp/etcd-download-test/etcdutl version
    ```

2. Executez les commandes suivantes pour ajouter les exécutables à la session.
    ```shell
    sudo mv /tmp/etcd-download-test/etcdctl /bin/etcdctl
    sudo mv /tmp/etcd-download-test/etcdutl /bin/etcdutl
    ```

> [!TIP]
> Pas de panic lors de la CKA, ces outils seront préinstallées 😉
