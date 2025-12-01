pipeline {
    agent none

    options {
        disableConcurrentBuilds()
        timestamps()
        buildDiscarder(logRotator(
            numToKeepStr: '10',
            artifactNumToKeepStr: '5'
        ))
    }

    environment {
        APP_NAME    = 'monitoringwebsekolah'
        VENV_DIR    = '.venv_ci'
        ARTIFACT    = "${APP_NAME}-${BUILD_NUMBER}.tar.gz"
        DEPLOY_ROOT = '/var/www/monitoringwebsekolah'  // folder utama di server prod
    }

    stages {

        stage('Checkout') {
            agent { label 'controller' }
            steps {
                checkout scm
                // KIRIM SEMUA KECUALI sampah; tests IKUT, tapi nanti TIDAK dimasukin artifact
                stash name: 'source',
                    includes: '**',
                    excludes: '.git/**, .venv/**, venv/**, __pycache__/**, runtime/**'
            }
        }

        stage('Install & Test') {
            agent { label 'stb' }
            steps {
                unstash 'source'
                sh """
                    set -e

                    python3 -m venv ${VENV_DIR} || true
                    . ${VENV_DIR}/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt

                    # TODO: tambahin pytest di sini kalau lu udah punya test beneran
                    # pytest -q || true
                """
            }
        }

        stage('Package Artifact') {
            agent { label 'stb' }
            steps {
                sh """
                    set -e
                    tar czf ${ARTIFACT} \\
                        app.py \\
                        requirements.txt \\
                        scripts \\
                        static \\
                        templates \\
                        reports \\
                        data \\
                        csvjson.json \\
                        notifications.json \\
                        onts.json \\
                        outages.json \\
                        history.json \\
                        ont_history.json \\
                        user_log.json \\
                        COLOR_LEGEND.md \\
                        COLOR_SYSTEM_DOCUMENTATION.md \\
                        QUICK_COLOR_REFERENCE.txt \\
                        TROUBLESHOOTING_PING.md || true
                """
                stash name: 'artifact', includes: "${ARTIFACT}"
                archiveArtifacts artifacts: "${ARTIFACT}", fingerprint: true
            }
        }

        stage('Deploy (PROD)') {
            agent { label 'prod' }
            steps {
                unstash 'artifact'
                sh """
                    set -e

                    SERVICE_WEB=monitoringwebsekolah
                    SERVICE_PING=pingsekolah

                    DEPLOY_ROOT="${DEPLOY_ROOT}"
                    RELEASES_DIR="\$DEPLOY_ROOT/releases"
                    CURRENT_LINK="\$DEPLOY_ROOT/current"

                    mkdir -p "\$RELEASES_DIR"

                    RELEASE_DIR="\$RELEASES_DIR/${APP_NAME}-${BUILD_NUMBER}"

                    # stop service kalau bisa, tapi JANGAN bikin build fail kalau sudo minta password
                    sudo -n systemctl stop "\$SERVICE_WEB" "\$SERVICE_PING" || true

                    rm -rf "\$RELEASE_DIR"
                    mkdir -p "\$RELEASE_DIR"
                    tar xzf "${ARTIFACT}" -C "\$RELEASE_DIR"

                    python3 -m venv "\$RELEASE_DIR/.venv"
                    . "\$RELEASE_DIR/.venv/bin/activate"
                    pip install --upgrade pip
                    pip install -r "\$RELEASE_DIR/requirements.txt"

                    ln -sfn "\$RELEASE_DIR" "\$CURRENT_LINK"

                    sudo -n systemctl daemon-reload || true
                    sudo -n systemctl restart "\$SERVICE_WEB" || true
                    sudo -n systemctl restart "\$SERVICE_PING" || true  

                    cd "\$RELEASES_DIR"

                    OLD_RELEASES=\$(ls -1dt ${APP_NAME}-* 2>/dev/null | tail -n +6 || true)

                    if [ -n "\$OLD_RELEASES" ]; then
                        echo "Menghapus release lama:"
                        echo "\$OLD_RELEASES" | tr ' ' '\\n'
                        echo "\$OLD_RELEASES" | xargs -r sudo -n rm -rf -- || true
                    fi
                """
            }
        }
    }
}
